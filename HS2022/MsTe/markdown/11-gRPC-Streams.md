# Streams

Performance-Thema in gRPC. 3 verschiedene Modi: Server > Client, Client > Server, Bidirektional.

End-To-End Reliability, garantiertes Ausliefern der Nachrichten
Ordered Delivery: Reihenfolge gewährleistet

Anwendungen: Messaging, Games, Live-Resultate, Smart Home Devices etc.

Synchrones Lesen vs. Streaming

## Protobuf Implementation

```protobuf
service FileStreamingService {
    rpc ReadFiles (google.protobuf.Empty)	
        returns (stream FileDto);			// Server Streaming Call
    rpc SendFiles (stream FileDto)			
        returns (google.protobuf.Empty);	// Client Streaming Call
    rpc RoundtripFiles (stream FileDto)
        returns (stream FileDto);			// Bidirektional
}
message FileDto {
	string file_name = 1;
	int32 line = 2;
	string content = 3;
}
```

## C# Implementation

### API Klassen

`AsyncServerStreamingCall<TResponse>`
`AsyncClientStreamingCall<TRequest, TResponse>`
`AsyncDuplexStreamingCall<TRequest, TResponse>`

Interfaces: `IAsyncStreamReader<T>`, `IClientStreamWriter<T>`



<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-streaming-api.png" style="zoom: 67%;" />

### Server Streaming Call

```csharp
// Service
public override async Task ReadFiles(
    Empty request, 
    IServerStreamWriter<FileDto> responseStream, 
    ServerCallContext context)
{
    string [] files = Directory.GetFiles(@"...");
    foreach (string file in files)
    {
        string content; int line = 0;
        using StreamReader reader = File.OpenText(file);
        while ((content = await reader.ReadLineAsync()) != null)
        {
            line++;
            FileDto reply = new()
            {
                FileName = file, Line = line, Content = content
            };
            await responseStream.WriteAsync(reply);
        }
    }
}
// Client
using AsyncServerStreamingCall<FileDto> call = client.ReadFiles(new Empty());
await foreach (FileDto message in call.ResponseStream.ReadAllAsync())
{
    WriteLine($"File: {message.FileName}, Line Nr: {message.Line}, Content: {message.Content}");
}
```

### Client Streaming Call

```csharp
//Service
public override async Task<Empty> SendFiles(IAsyncStreamReader<FileDto> requestStream, ServerCallContext context)
{
	await foreach (FileDto message in requestStream.ReadAllAsync())
	{ WriteLine($"File: {message.FileName}, Line Nr: {message.Line}, Line Content: {message.Content}");	}
	return new Empty();
}
// Client
using AsyncClientStreamingCall<FileDto, Empty> call = client.SendFiles();
string[] files = Directory.GetFiles(@"Files");
foreach (string file in files)
{
    string content; int line = 0;
    using StreamReader reader = File.OpenText(file);
    while ((content = await reader.ReadLineAsync()) != null)
    {
        line++;
        FileDto reply = new() { FileName = file, Line = line, Content = content };
        await call.RequestStream.WriteAsync(reply);
    }
}
// Required!
await call.RequestStream.CompleteAsync(); // No more messages to come (server exits foreach-Loop)
Empty result = await call; // Wait until service method is terminated / Get the result
```

### Bidirectional / Duplex Streaming Call

```csharp
// Service
public override async Task RoundtripFiles(
    IAsyncStreamReader<FileDto> requestStream,
    IServerStreamWriter<FileDto> responseStream,
    ServerCallContext context)
{
    await foreach (FileDto message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(message);
        WriteLine($"File: {message.FileName}, Line Nr: {message.Line}, Line Content: {message.Content}");
    }
}
// Client
using AsyncDuplexStreamingCall<FileDto, FileDto> call = client.RoundtripFiles();
// Read
Task readTask = Task.Run(async () =>
{
    await foreach (FileDto message in call.ResponseStream.ReadAllAsync())
    {
        WriteLine($"File: {message.FileName}, Line Nr: {message.Line}, Line Content: {message.Content}");
    }
});
// Write
string[] files = Directory.GetFiles(@"Files");
foreach (string file in files)
{
    string content; int line = 0;
    using StreamReader reader = File.OpenText(file);
    while ((content = await reader.ReadLineAsync()) != null)
    {
        line++;
        FileDto reply = new()
        { FileName = file, Line = line, Content = content };
        await call.RequestStream.WriteAsync(reply);
    }
}
// Required!
await call.RequestStream.CompleteAsync(); // No more messages to come (server exits foreach-Loop)
await readTask; // Wait until service method is terminated / all messages are received by clien
```
