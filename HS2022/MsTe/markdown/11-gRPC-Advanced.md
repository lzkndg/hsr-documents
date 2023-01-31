# gRPC Advanced Topics

## Exception Handling

Weitergabe einer Exception vom Server auf den Client via gRPC nicht einfach so möglich: aus Sicherheitsgründen, wegen fehlender Serialisierungslogik einer Exception, wegen möglicherweise unterschiedlicher Programmiersprachen von Server und Client.

```csharp
public class RpcException : Exception
{
    public RpcException(Status status);
    public RpcException(Status status, string message);
    public RpcException(Status status, Metadata trailers);
    public RpcException(Status status, Metadata trailers, string message);
    
    public Status Status { get; }
    public StatusCode StatusCode { get; }
    public Metadata Trailers { get; }		// Möglichkeit, weitere Details an den Client zu übermitteln
}
public struct Status
{
    public static readonly Status DefaultSuccess = new Status(StatusCode.OK, "");
    public static readonly Status DefaultCancelled = new Status(StatusCode.Cancelled, "");
    public Status(StatusCode statusCode, string detail);
    public StatusCode StatusCode { get; }
    public string Detail { get; }
    public override string ToString();
}
```

Standard-Mechanismus auf Übertragungskanal: Nutzung von **HTTP Status Codes**. Convenience Features seitens .NET: **RpcException**. Jede mögliche Exception innerhalb des Servers sollte in eine RpcException verwandelt werden. Somit kann die .NET gRPC Implementation diese in eine saubere HTTP Response verwandeln.

| Status Code        | Beschreibung                                                 |
| ------------------ | ------------------------------------------------------------ |
| OK                 | Kein Fehler, alles okay.                                     |
| Cancelled          | Operation wurde abgebrochen (meist durch Client).            |
| Unknown            | Fehler kann nicht ermittelt werden / ist unbekannt. Default wenn Exception nicht behandelt wird. |
| InvalidArgument    | Ungültige Argumente beim Aufruf angegeben.                   |
| DeadlineExceeded   | Deadline überschritten bevor Anfrage erfolgreich beendet wurde. |
| NotFound           | Angefragte Ressource wurde nicht gefunden.                   |
| AlreadyExists      | Zu erstellende Ressource existiert bereits.                  |
| PermissionDenied   | Aufrufer hat keine Berechtigung um die Operation auszuführen. |
| Unauthenticated    | Aufrufer ist nicht authentifiziert (angemeldet).             |
| ResourceExhausted  | Eine Ressource ist aufgebraucht (Per-User-Quota / Speicherplatz / etc.) |
| FailedPrecondition | Vorbedingungen sind falsch (ungültiger Systemstatus, keine Datenbankverbindung, Bestellung abschliessen obwohl bereits abgeschlossen). |
| Aborted            | Anfrage wurde abgebrochen (Concurrency Issues, Transaktionsabbruch, etc.). |
| OutOfRange         | Ungültiger Range bei Anfrage (Geburtsdatum in der Zukunft, etc.). |
| Unimplemented      | Methode wurde (noch) nicht implementiert.                    |
| Internal           | Allgemeiner interner Serverfehler                            |
| Unavailable        | Service ist aktuell nicht verfügbar.                         |
| DataLoss           | Datenverlust oder korrupte Daten                             |

Wird von der Server Runtime eine nicht-RPC-Exception gefangen, wird diese allgemein in eine StatusCode Unknown RPC-Exception umgewandelt mit leerem Trailer. Eine RPC-Exception wird jeweils direkt übertragen.

```csharp
public override Task<Empty> NotFound(Empty request, ServerCallContext context)
{
    throw new RpcException(
        new Status(
        StatusCode.NotFound, "Something was not found.")
        //Optional: Trailers mitgeben 
        new Metadata
        {
            { "error-details", "Here are some more Details" },
            { "error-obj" + Metadata.BinaryHeaderSuffix, Encoding.UTF8.GetBytes("..payload..") }
            // Metadata.BinaryHeaderSuffix fügt dem Namen "-bin" hinzu für binary-Data
        }
	);
}
```

### Common Mistakes

Unimplemented kann heissen: 

- Service in Startup nicht registriert mit `endpoints.MapGrpcService<...>();`
- Methode in Proto File definiert aber nicht implementiert (überschrieben).

Unknown kann auf einen fehlenden Catch-Block für aufgetretene Fehler hindeuten.

### Clientseite

Exceptions können mit when-Klauseln abgefangen werden:

```csharp
try { /* gRPC call(s) */ }
// Communication exceptions
catch (RpcException e)
	when (e.StatusCode == StatusCode.Unavailable) {}
catch (RpcException e)
	when (e.StatusCode == StatusCode.NotFound) {}
catch (RpcException e)
	when (e.StatusCode == StatusCode.Aborted) {}
catch (RpcException e) {}
// Other stuff
catch (Exception) { /* ... */ }
```

## Well Known Types

| Typ        | Beschreibung                                                 |
| ---------- | ------------------------------------------------------------ |
| Emtpy      | `import "google/protobuf/empty.proto"`                       |
| Timestamp  | `import "google/protobuf/timestamp.proto"`, immer auf UTC ohne daylight saving bezogen. |
| ByteString | Typ `bytes` ohne Import in proto Files verwendbar. in C#: `var bs = ByteString.CopyFrom("X", Encoding.Unicode);` |

### Repeated Fields

```protobuf
message RepeatedResponse { repeated string results = 1; }
```

```csharp
RepeatedResponse response = new();
response.results.Add("Hello");
string [] arr = { "A", "B" }
response.results.AddRange(arr);
```

### Map Fields

```protobuf
message MapResponse { map<int32, string> results = 1 }
```

```csharp
MapResponse response = new();
response.results.Add(1, "Hello"); // Single Add
// Verwendung wie ein IDictionary 
bool exists = response.results.ContainsKey(1);
string result= response.results[1];
```

### OneOf

Lässt eine Auswahl von Typen zu, liefert aber immer nur einen davon zurück. Logik in generierter Klasse implementiert, dass immer nur einer der beiden Werte gesetzt sein kann.

```protobuf
message OneofResponse {
    oneof results {
        string image_url = 1;
        bytes image_data = 2;
    }
}
```

```csharp
OneofResponse response = new() { ImageUrl = "http://..." };
var rc = response.ResultsCase;		// ImageUrl -> wird in C# als Enum implementiert
string s = response.ImageUrl; 		// "http://..."
ByteString bs = response.ImageData;  // default
```

### Any

```protobuf
import "google/protobuf/any.proto"
message AnyResponse { google.protobuf.Any results = 1; }
```

```csharp
AnyResponse = new();	// Response Objekt
response.Results = Any.Pack(new CustomerResponse()); // Objekt befüllen

bool isCust = response.Results.Is(CustomerResponse.Descriptor);
bool success = response.Results.TryUnpack(out CustomerResponse parsed); // safe unpack
response.Results.Unpack<CustomerResponse>(); // unsafe unpack

```

## Configuration und Logging

### Serverseitig

| Option                | Default | Beschreibung                                                 |
| --------------------- | ------- | ------------------------------------------------------------ |
| MaxSendMessageSize    | null    | Maximale Message-Grösse beim Senden (Exception falls grösser). |
| MaxReceiveMessageSize | 4 MB    | Maximale Message-Grösse beim Empfangen (Exception falls grösser). |
| EnableDetailedErrors  | false   | Wenn true werden Details zu Exceptions an den Client übermittelt. Achtung: Möglicher Security Flaw |
| CompressionProviders  | gzip    | Kompressions-Algorithmus                                     |

```csharp
.Services
    .AddGrpc(options => // Global
    {
        options.MaxSendMessageSize = 5 * 1024 * 1024; // 5 MB
        options.MaxReceiveMessageSize = 2 * 1024 * 1024; // 2 MB
        options.EnableDetailedErrors = true;
        options.CompressionProviders = new List<ICompressionProvider> { new 												GzipCompressionProvider(CompressionLevel.Optimal) };
})
.AddServiceOptions<AdvancedService>(options => // Specific Service
{
    options.MaxSendMessageSize = 1 * 1024 * 1024; // 1 MB
});
```

### Clientseitig

| Option                | Default | Beschreibung                                                 |
| --------------------- | ------- | ------------------------------------------------------------ |
| HttpClient            | Objekt  | HttpClient für gRPC Verbindung. Kann verändert werden z.B. um weitere HTTP Headers zu schicken. |
| DisposeHttpClient     | false   | Wenn true und Verwendung eines Custom HttpClient (siehe oben) wird der Client mit dem gRPC Channel disposed |
| LoggerFactory         | null    | Logger Factory für gRPC Calls                                |
| MaxSendMessageSize    | null    | Maximale Message-Grösse beim Senden (Exception falls grösser). |
| MaxReceiveMessageSize | 4MB     | Maximale Message-Grösse beim Empfangen (Exception falls grösser). |
| Credentials           | null    | Credentials für die Authentifizierung am Server              |
| CompressionProviders  | gzip    | Kompressions-Algorithmus                                     |

```csharp
GrpcChannel channel = GrpcChannel.ForAddress("https://localhost:5001",
	new GrpcChannelOptions
	{
        MaxSendMessageSize = 2 * 1024 * 1024, // 2 MB
        MaxReceiveMessageSize = 5 * 1024 * 1024, // 5 MB
        // ...
    }
);
```

## Logging

via appsettings.json

```json
{
    "Logging": {
    "LogLevel": {
        "Default": "Warning",
        "Microsoft.Hosting.Lifetime": "Information",
        "Grpc": "Debug"
	}
},
    "AllowedHosts": "*",
    "Kestrel": {
        "EndpointDefaults": {
            "Protocols": "Http2"
        }
    }
}
```

via Program.cs

```csharp
builder.Logging.AddFilter("Grpc", LogLevel.Debug);
```
