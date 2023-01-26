# Delegates, Events, Lambdas

## "Normale" Delegates

Verwendungszwecke: Methoden als Funktionsparameter, Callback / Observer Pattern

Delegates sind typsichere Funktionspointer, abgeleitet von `System.Delegate`, somit eine Referenz auf 0-n Methoden. Analog Function Pointers in C++. Deklaration geschieht auf Namespace-Level.
Jeder Delegate ird vom Compiler als Klasse generiert (abgeleitet von `MulticastDelegate`) und kann danach instanziert und zugewiesen werden. Null-Zuweisung ist möglich, wirft bei Ausführung NullReferenceException. Inversion of Control, Möglichkeit auch mit Interfaces. 

*Delegates sind eine explizite Vereinfachung von Interfaces mit genau einer Methode.* Zuweisung einer Methode zur Delegate Variable entspricht der Instanzierung eines Interfaces.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-delegate-interface.png" style="zoom:50%;" />

Eine Variable vom Typ Delegate hat 3 Felder

- `Target`: Klasseninstanz, wenn die angegebene Methode nicht statisch ist. 
- `Method`: Methode die zugewiesen wurde / ausgeführt wird
- `Prev`: siehe Multicast Delegates

```csharp
// Verwendung von Delegate zur Übergabe von Funktionen als Methodenparameter
public delegate void Comparer(object x, object y);
class Program {
    static int CompareFraction(object x, object y) { ... }
	static int CompareString(object x, object y)
    { return ( (string)x ).CompareTo(y); }
    
    static void Sort(object[] a, Comparer compare) { /* do some comparison using delegate */ }
    Comparer cf = CompareFraction;			// Methode zu Delegate zuweisen
    Fraction[] fractions = new Fraction[] { new(1, 2), new(3,4) };
    Sort(fractions, cf)						// Delegate übergeben
}
```

Wenn die Methode in derselben Klasse ist wie die Delegate Variable, ist `obj` "this" und kann weggelassen werden. 
Bei statischer Methode der Klassenname.

```csharp
// Zuweisungs-Syntax
DelegateType delegateVar = obj.Method;
```

Zugewiesene Methode 

- darf nicht abstract sein
- darf virtual, override und/oder new sein
- muss identisch sein in der Signatur (inkl. Anzahl, Art und Typen der Parameter, Rückgabetyp) wie im Delegate definiert

```csharp
// Aufruf-Syntax
object result = delegateVar[.Invoke](params); 		// Mit Rückgabetyp
if (delegateVar != null) { delegateVar(params); }	// Mit Null-Check
delgegateVar?.Invoke(params);					  // Standard-Variante ab C# 6.0 mit Null-Check
```

## Multicast Delegates

Mehrere registrierte Methoden werden synchron in der registrierten Reihenfolge ausgeführt. Exception führt zum Abbruch aller restlichen Methoden. Invoke-Methode regelt die Ausführung der jeweiligen Zielmethode.

`Prev` Feld: Referenz auf vorheriges (Multicast-)Delegate aus der Linked List.

```csharp
Notifier greetings;
greetings = SayHi;
greetings += SayCiao; // fügt eine weitere Methode der Ausführungskette hinzu
greetings -= SayHi; // sucht "von hinten" in der linked List nach dem ersten Match und entfernt ihn.
Notifier a = SayHi; Notifier b = SayCiao;
Notifier c = a + b; // (Notifier)Delegate.Combine(a, b);
```

Compiler wandelt jeden "normal" definierten Delegate in eine eigene Klasse um, die von MulticastDelegate ableitet. Bei Methoden mit Rückgabewerten wird jeweils nur der letzte Wert behalten.

## Events

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-events.png" style="zoom: 35%;" />

Button: Observable / Usage: Observer

Definiert wird immer Event sowie ein Delegate, der die Struktur der Methode angibt, welche auf den Event hin aufgerufen wird (bzw. zum Event Handling angemeldet werden kann).

Der Delegate hat standardmässig `-Handler` im Namen und befindet sich wiederum auf Namespace Level.
Der Event gibt an, welchen Delegate er "verwendet" und wird an bestimmten Stellen im Code via EventName?.Invoke(params) ausgelöst.

```csharp
public delegate void TickEventHandler (int ticks, int interval);
public class Clock
{
    // Selber definierter Event
    public event TickEventHandler OnTickEvent;
    // Compiler-generierter Output
    private TickEventHandler OnTickEvent;		// Delegate Instanzierung wird private
    // Subscribe/Unsubscribe Logik wird public generiert, Zugriff via Event (+=/-=)
    public void add_OnTickEvent(TickEventHandler h) { OnTickEvent += h; }
    public void remove_OnTickEvent(TickEventHandler) { OnTickEvent -= h; }
}
```

Standardmässige Delegate Parameter 

- `sender` Absender übergibt bei Aufruf des Delegates / Events «this» mit. In UIs meist ein Button oder ähnliches.
- `EventArgs`: Argumente des Events werden als Instanz einer von EventArgs abgeleiteten Klasse übergeben, falls benötigt. Klasse enthält 0-n Properties, die weitere Angaben zum Event machen. Beispiel: welche Maustaste beim Click, welche Zeit, welche Position... 

```csharp
public delegate void ClickEventHandler(object sender, AnyEventArgs e);

public class ClickEventArgs : EventArgs /* kann beliebig um weitere parameter ergänzt werden*/ 
{ 
    public string MouseButton { get; set; } 
}

public class Button 
{ 
    public event ClickEventHandler OnClick; // "Returntype" definiert Delegate, der den Event "handeln" kann
}
// Verwendung
public void Test() { Button b = new(); b.OnClick += OnClick; }
private void OnClick(object sender, ClickEventArgs eventArgs) { /* do something */ }
```

Zur Verwendung im "Client Code" muss nur eine entsprechende Methode die zum Delegate passt erstellt werden, und diese auf den Event einer Observable-Instanz registriert werden.

## Lambda Expressions

Zuweisung von Methoden zu Delegates oder Events müssen nicht zwingend benannt sein, die entsprechenden Methoden können als Lambdas In-Place definiert werden.

```csharp
Func<int, bool> fe = i => i % 2 == 0; 	// Expression Lambda: "Func<param, param, .., return type>"
Func<int, bool> fs = i => 
{
    int rest = i%2;
    return rest == 0;
}									// Statement Lambda, mehrere Zeilen mit { } eingepackt
```

Varianten von Parametern

```csharp
p01 = () => true; 					 // Kein Parameter
p02 = (a) => true; p02 = a => true;    // Ein Parameter (Klammer optional)
p03 = (a, b) => true; 				  // etc...
p04 = (ref int a) => true; 			  // Ref oder Out Parameter auch möglich
```

Statement Lambdas: theoretisch unbegrenzte Anzahl Statements, idealerweise aber nicht mehr als 2-3 verwenden. Sonst besser eine named Methode inkl. Tests.

<img src="C:\Users\luzia\OneDrive - Ost\01_Aktuelles_Semester\MsTe\res\csharp-func-delegate.png" alt="Func Delegate" style="zoom: 50%;" />

## Closure

Problem: Lambda Expression hat grundsätzlich keinen Zugriff auf die Variablen der umgebenden Methode/Klasse.

Lösung Closure: Compiler generiert eine Hilfsklasse, welche die Logik des Delegates enthält, inklusive aller Variablen, auf die das Delegate zugreift. 
