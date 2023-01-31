# Tasks, Async / Await

## Tasks

Platzhalter für ein Resultat, welches noch nicht bekannt ist, hat einen Rückgabewert. Repräsentiert eine asynchrone Operation. Convienence Wrapper um einen C# Thread (= 1 OS Thread) herum. Vereinfacht Programmfluss mit async/await.

```cs
Task<int> t1 = Task.Factory.StartNew(() => { Thread.Sleep(2000); return 1; });
Task<int> t2 = Task.Run(() => { Thread.Sleep(2000); return 1; });
```

### Synchrone Waits

Return aus der Methode nachdem die gesamte Logik durchlaufen wurde. Blockieren den Aufrufer / aktuellen Thread (Problematisch in GUI Applikationen!). 
Beispiele: `t1.Wait(), Task.WaitAll(t1, t2), t1.Result, t1.GetAwaiter().GetResult()`

```csharp
while (!t1.IsCompleted)				// Busy Wait (bad Idea!)
{	/* Do some stuff */ } 
int result1 = t1.Result;			
t1.Wait(); Task.WaitAll(t1, t2);	// Explicit Wait
int result2 = t1.Result;
int result3 = t1.GetAwaiter().GetResult(); // Awaiter, optimierteres Exception Handling
```

### Asynchrone Waits

Ruft eine Methode auf, ohne auf das Resultat zu warten. Möglichkeit zur Benachrichtigung via Callbacks, oder Rückgabe eines `Task`-Objektes das abgefragt werden kann.

_Continuations_ werden erst dann ausgeführt, wenn das Resultat vorhanden ist. Sind somit nicht-blockierend.

```cs
Task<int> t1 = GetSomeCustomerIdAsync();
t1.ContinueWith(id =>			// id: Result Wert vom Task, sobald vorhanden...
{
    Task<string> t2 = GetOrdersAsync(id.Result);	// nicht mehr blockierend

    t2.ContinueWith(order =>	// order: Result Wert vom Task, sobald vorhanden...
        Console.WriteLine(order.Result)				// nicht mehr blockierend
    );
});
```

Merke: Asynchron nutzt denselben Thread für verschiedene Aufgaben. Parallel nutzt verschiedene Threads.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-async-parallel.png" style="zoom:50%;" />

_Frühstück asynchron kochen heisst, mögliche Wartezeit für die nächste Aufgabe zu nutzen. Parallel kochen heisst, 3 Köche stehen vor einer anderen Pfanne und warten. Geht auch nicht schneller..._

## async/await

Keyword `async` markiert eine Methode als asynchron. Rückgabetypen sind dabei eingeschränkt auf `Task` (Methode ohne Rückgabewert), `Task<T>` (Methode mit Rückgabewert vom Typ `T`), oder void (fire and forget). 

Alles was nach dem Keyword `await` folgt, wird vom Compiler in eine Continuation umgewandelt. Keyword ist nur in async-Methoden erlaubt. 

`Task.WhenAll(t1, t2)`  wandelt das abwarten von mehreren Tasks in einen einzigen Task um, der erst beendet ist, wenn alle Subtasks auch fertig sind:

```csharp
string[] allResults = await Task.WhenAll(t1, t2)
```

Overhead für Task bei ca 10ms (?) -> lohnt sich erst bei grösseren Tasks.

.. eine asynchrone Methode (computePiAsync()) zu implementieren, bedeutet NICHT dass die Methode an sich _async_ sein muss. Sie gibt einfach einen Task zurück.

## Cancellation Support

Integriertes Programmiermodell für das Abbrechen von lange dauernder Programmlogik. Ein CancellationToken wird durch die gesamte Aufrufkette durchgereicht. Ist bei jeder asynchrone Methode der letzte Parameter (Best Practice).

Kann erstellt werden über die Klasse `CancellationTokenSource`. Wenn der Abbruch via Source ausgelöst wird, wird für alle ihre Tokens ein Abbruch angefordert.

```csharp
CancellationTokenSource cts = new(); // Source wird instanziert
CancellationToken ct1 = cts.Token; // Emittierung eines Token via Property
cts.Cancel(); 					 // Bricht alle laufenden Operationen ab
```

Beispiel: 

```csharp
CancellationTokenSource cts = new();
CancellationToken ct  = cts.Token;
Task t1 = LongRunning(1000, ct);
Task t2 = LongRunning(2000, ct);
Task t3 = Longrunning(3000, default); // independent, not cancellable

await Task.Delay(2000, ct);
cts.Cancel();

async Task LongRunning(int ms, CancellationToken ct)
{
    Console.WriteLine($"{ms}ms Task started.");
    await Task.Delay(ms, ct);
    Console.WriteLine($"{ms}ms Task completed.");    
}
```

Auslösen von `cts.Cancel()` wird meist mit Ctrl-C auf der Konsole verknüpft.

```csharp
Console.CancelKeyPress += (_, e) =>
{
    cts.Cancel();
    e.Cancel = true;
};
```

