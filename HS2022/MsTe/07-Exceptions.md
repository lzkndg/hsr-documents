# Exceptions

Behandeln unerwartete Programmzustände oder Ausnahmeverhalten zur Laufzeit. Best Practices dazu sind

- Wenn möglich Vorbedingungen prüfen um Exceptions zu vermeiden
- Exceptions sind "Fehlercodes" vorzuziehen
- Konkrete Fehlerbeschreibung: möglichst konkrete Klassen verwenden, möglichst detaillierte Beschreibung des Fehlers
- Fehlerbeschreibung NIE über "Web-Schnittstellen" übermitteln, offenbart Internas und erhöht die Verletzbarkeit.
- Wichtig: Aufräumen nach Exceptions! (offene Sockets, File Handles, Transaktionen etc.)

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-exceptions.png" style="zoom:50%;" />

## Beispiel / Catch

```csharp
try { ... something ... }
catch (FileNotFoundException e)					// Exception inklusive Name wenn verwendet
{ Console.WriteLine("{0} not found", e.FileName) }
catch (IOException)								// Exception nur mit Typ
{ Console.WriteLine("IO exception occurred") }
catch										  // Jegliche Exception
{ Console.WriteLine("Unknown error occurred") }
finally
{ /* Wird immer ausgeführt */ }
```

## Basisklasse System.Exception

```csharp
public class Exception: ISerializable, _Exception
{
    public Exception(); 
    public Exception(string message); 
    public Exception(string message, Exception innerException);
    
    public Exception InnerException { get; }	// Verschachtelung von Exceptions möglich
    public virtual string Message { get; }		// Fehlermeldung
    public virtual string Source { get; set; }	// Name der Applikation, Objekt, Framework 
    public virtual string StackTrace { get; set; }	// Methodenaufrufkette als String
    public MethodBase TargetSite { get; }		// Ausgeführter Codeteil, der den Fehler verursacht
    
    public override string ToString();			// Fehlermeldung und StackTrace als String
}
```

## Throw

Werfen einer Exception, geschieht implizit bei ungültigen Operationen, bei Methodenaufrufen welche eine unbehandelte Exception werfen oder explizit mittels throw-Statement.

```csharp
throw new Exception("An error occurred");

try { ... }
catch (Exception e)
{ throw e; } 	// beginnt neuen Stack Trace
{ throw; } 		// Strack Trace bleibt erhalten
```

## Suche nach Catch-Klausel

Call stack wird rückwärts nach passender Catch Klausel durchsucht. Programmabbruch mit Fehlermeldung und Stack-Trace, falls keine Passende gefunden. (Multicast-) Delegates werden dabei wie normale Methoden behandelt, die nacheinander ausgeführt werden.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-exceptions-callstack.png" style="zoom:50%;" />

## Exception Filters

Catch-Block kann auch nur unter definierten Bedingungen ausgeführt werden. `when`-Klausel erwartet eine Boolean Expression.

```csharp
try { Console.WriteLine("Drink some beer") }
catch (Exception e) when (DateTime.Now.Hour < 16) { /* Kein Bier vor vier! */ }
```

## Unchecked Exceptions

In C# kann jede Exception auch unbehandelt bleiben. Java benötigt dazu `throws`-Klausel in Methodensignatur.

## Argumente prüfen

Refactoring-stabile Variante um Argumente auf Null-Werte oder Out Of Range zu prüfen:

```csharp
string Replicate(string s, int nTimes)
{
    if (s == null) 
        throw new ArgumentNullException(nameof(s));
}
```

## First Chance Exception

Debugger wird genau an der Stelle angehalten, wo die Exception auftritt. Standardmässig sonst dort, wo das Programm abstürzt.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-exceptions-firstchance.png" style="zoom: 25%;" />