# gRPC

## Überblick

Alternative zu REST: ProtoBuf wird eher für Server-to-Server Kommunikation verwendet (z.B. für Microservices). Fokus dabei ist hohe Performance. Als Frontend-API bessere Alternativen sind REST oder GraphQL. ProtoBuf wird auch als Replacement für Windows Communication Foundation WCF verwendet.

Basistechnologien:

- HTTP/2
- Interface Definition Language (IDL): Google Protocol Buffers

### Remote Procedure Call

Auch RPC, ermöglicht Client-Server Kommunikation. Wird in fast allen verteilten Systemen verwendet. 
Einfacher TCP Call kennt das Konzept einer Methode (Procedure) nicht.

Wird in fast allen verteilten Systemen verwendet. Verschiedene Implementationen: WCF, Cobra, JSON-RPC, SOAP, NFS, Apache Thrift / Avro, D-Bus.

### gRPC Grundprinzipien

- Einfache Service Definition (IDL, Sprachunabhängig)
- Problemlos skalierbar (stateless)
- Bi-direktionales Streaming
- Integrierte Authentisierungsmechanismen
- Unterstützt verschiedenste Sprachen (Java, .NET, C++, Dart, Go, Kotlin, Node, Python, ...)

### Developer Workflow

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\grpc-developer-workflow.png" style="zoom:50%;" />

## Architektur

gRPC ist ein Software Development Kit. Plattform- und Sprachneutral. Visual Studio (Code) Integration bietet Features wie automatische Code Generierung, Intelli-Sense sowie Debugging & Logging. Kommunikation über HTTP/2.

### HTTP/2

Ermöglicht Multiplexing von HTTP Calls pro TCP/IP Connection, verringert Overhead für Disconnect/Reconnect massiv (8 Connections Limit oft vom Browser vorgegeben). 
Bidirektionales Streaming ermöglicht paralleles Senden und Empfangen. 
HTTP/2 und gRPC basieren vollständig auf HTTPS, Kommunikation ist also standardmässig verschlüsselt.
Header Compression optimiert die zu übertragenden Daten nocheinmal.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\grpc-vergleich-rest.png" style="zoom:50%;" />

## Protocol Buffers (ProtoBuf) Basics

### Umfang

- IDL: Subform einer Domain Specific Language. Beschreibt ein Service Interface
- Data Model: Beschreibt Messages bzw. Request- und Response Objekte
- Wire Format: Beschreibt das Binärformat zur Übertragung
- Serialisierungs- und Deserialisierungsmechanismen
- Service-Versionierung

### Wertetypen

| .proto | C#         | Default | Beschreibung                                                 |
| ------ | ---------- | ------- | ------------------------------------------------------------ |
| double | double     | 0       |                                                              |
| float  | float      | 0       |                                                              |
| int32  | int        | 0       | Variable-lengt encoding. Ineficcient for negative numbers (use sint32 instead) |
| int64  | long       | 0       | Variable-lengt encoding. Ineficcient for negative numbers (use sint64 instead) |
| bool   | bool       | false   |                                                              |
| string | string     | <empty> | String must always contain UTF-8 encoded or 7-bit ASCII text. |
| bytes  | ByteString | <empty> | May contain an arbitrary sequence of bytes.                  |
|        |            |         | weitere wie sint32/64, uint32/64, fixed32/64                 |

### Proto-Files

Hat jeweils die Dateiendung `.proto`. Das File enthält im Header allgemeine Informationen. Der Inhalt besteht anschliessend aus 0-n Services und 1 oder mehr Service-Methoden (Messages) pro Service. 
Die Felder eines Message Types beinhalten jeweils:

- `Typ`: Skalarer Wertetyp, anderer Message Type, Enumeration
- `Unique Name`: Lower Snake Case
- `Unique Field Number`: Zur Versionierung bzw. als Identifikator für das Binärformat. Wertebereich  1 - ca. 500'000, mit Ausnahme: 19'000-19'999, diese sind vom Protokoll reserviert.

```protobuf
syntax = "proto3";
option csharp_namespace = "_01_BasicExample"; // verwendet für generierten C# Code
package Greet;						// Package-Name
import "protos/_base.proto"			 // Message Types können wiederverwendet werden

service Greeter {					// Service
    rpc SayHello (HelloRequest)			// Service Methode
        returns (HelloReply);
}
message HelloRequest {				// Message Type
	string name = 1;					// Field		(lower snake case)
	repeated string words = 1;			 // Repeated Field: Liste von Werten
}
message HelloReply {
	string message = 1;
	reserved 1, 3, 20 to 30;			// Reservierte Versionierung
	reserved "page_number";				// Reservierte Feldnamen, für spätere Verwendung
}
enum Color {						// Enumeration: kann entweder in einer Message oder im Root des Files 
	RED = 0;						// definiert werden. Wert 0 ist zwingend = default value.
	GREEN = 1;
}
```

Service Methoden beinhalten immer nur genau 1 Parameter und 1 Rückgabetyp. Leere Werte werden mit `google.protobuf.Empty` definiert (benötigt `import "google/protobuf/empty.proto"`).

### Advanced Types



## Verwendung

Compiler: `protoc.exe` ist Bestandteil vom NugetPackage `Grpc.Tools`, zusammen mit einigen Plugins zur Generierung von C# Code aus `.proto`-Files. Damit wird der Proto Compiler direkt von Visual Studio in die Build Chain eingefügt.

Proto-Compiler Output in `<projectRoot>\obj\Debug\netcoreappX.X`, nützlich bei Build Fehlern.

Einbindung im `.csproj` File von Server und Client Code, Einstellungen möglich im Visual Studio bei Klick auf das Proto-File, unter Properties (F4).

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    <PropertyGroup>
        <TargetFramework>netcoreapp3.0</TargetFramework>
        <RootNamespace>BasicExample</RootNamespace>
    </PropertyGroup>
    <ItemGroup>
        <ProtoBuf Include="<relative-path>/greet.proto" GrpcServices="[Server|Client]">
            <Link>Protos/greet.proto</Link> <!-- Ort der Verlinkung -->
        </ProtoBuf>
    </ItemGroup>
    <ItemGroup>
        <PackageReference  Include="Grpc.AspNetCore" Version=".." /> <!-- Server -->
        <PackageReference  Include="Google.Protobuf" Version=".." /> <!-- Client -->
        <PackageReference  Include="Grpc.Net.Client" Version=".." /> <!-- Client -->
        <PackageReference  Include="Grpc.Tools" Version=".." /> <!-- Client -->
    </ItemGroup>
</Project>
```

**Serverprojekt**: Automatisch vom Typ ASP.NET Core Projekt, mit `dotnet new grpc` oder via Visual Studio.  Packages: `Grpc.AspNetCore(.Server)` 
**Clientprojekt**: Beliebiger Typ, Proto-File als Kopie oder Link eingebunden. Packages: `Grpc.Net.Client`, `Google.Protobuf`, `Grpc.Tools`

### Generierter Code

Pro Service wird eine abstrakte Basisklasse erzeugt:

```csharp
public static partial class Greeter { // Gründe unklar für die äussere Klasse
    public abstract partial class GreeterBase { /* ... */ } // <-- zu verwenden!
}
```

Manuell zwingend: Implementieren von allen Methoden der statischen Basisklasse.

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        => Task.FromResult(new HelloReply { Message = "Hello " + request.Name } );
}
```

Server: Service muss beim Startup registriert werden: 

```csharp
WebApplicationBuilder builder = WebApplication.CreateBuilder(args); // ASP.NET spezifisch
builder.Services.AddGrpc(); 			// GRPC für Dependency Injection konfigurieren
WebApplication app = builder.Build();
app.MapGrpcService<GreeterService>();    // Registrieren jedes Service
```

## Beispiel: Customer Service

### Proto-File

```protobuf
// Customers
service CustomerService {
	rpc GetCustomers (google.protobuf.Empty)
		returns (GetCustomersResponse);
	rpc GetCustomer (GetCustomerRequest)
		returns (GetCustomerResponse);
}

message GetCustomersResponse {
	repeated CustomerResponse data = 1;
}
message GetCustomerResponse {
	CustomerResponse data = 1;
}
message GetCustomerRequest {
	int32 id_filter = 1;
	bool include_orders = 2;
}
message CustomerResponse {
    int32 id = 1;
    string first_name = 2;
    string last_name = 3;
    Gender gender = 4;
    repeated OrderResponse orders = 10;
}
enum Gender { UNKNOWN = 0; FEMALE = 1; MALE = 2; }
// Orders
service OrderService {
	rpc GetOrders (GetOrdersRequest)
		returns (GetOrdersResponse);
}
message GetOrdersRequest {
	int32 customer_id_filter = 1;
}
	message GetOrdersResponse {
	repeated OrderResponse data = 1;
}
    message OrderResponse {
    string product_name = 1;
    int32 quantity = 2;
    double price = 3;
}
```

### CustomerService

```csharp
public class MyCustomerService : CustomerService.CustomerServiceBase
{
	public override Task<GetCustomersResponse> GetCustomers(
		Empty request, ServerCallContext context)
	{ /* ... */ }
	public override Task<GetCustomerResponse> GetCustomer(
		GetCustomerRequest request, ServerCallContext context)
	{ /* ... */ }
}
```

### OrderService

```csharp
public class MyOrderService : OrderService.OrderServiceBase
{
    public override Task<GetOrdersResponse> GetOrders(
	    GetOrdersRequest request, ServerCallContext context)
    { /* ... */ }
}
```

### Client-Implementation

```csharp
// The port number (5001) must match the port of the gRPC server.
GrpcChannel channel = GrpcChannel.ForAddress("https://localhost:5001");

// Customer service calls
CustomerService.CustomerServiceClient customerClient = new(channel);

Empty request1 = new();
GetCustomersResponse response1 = await customerClient.GetCustomersAsync(request1);
Console.WriteLine(response1);

GetCustomerRequest request2 = new() { IdFilter = 1 };
GetCustomerResponse response2 = await customerClient.GetCustomerAsync(request2);
Console.WriteLine(response2);

request2.IncludeOrders = true;
response2 = await customerClient.GetCustomerAsync(request2);
Console.WriteLine(response2);

// Order service calls
OrderService.OrderServiceClient orderClient = new(channel);

GetOrdersRequest request3 = new() { CustomerIdFilter = 1 };
GetOrdersResponse response3 = await orderClient.GetOrdersAsync(request3);
Console.WriteLine(response3);
```
