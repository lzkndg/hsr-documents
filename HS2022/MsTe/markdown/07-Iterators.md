# Iteratoren

Iterator/Enumerator und Iterable/Enumerable bedeutet jeweils dasselbe. 

Zum Iterieren über Collections werden for each Loops verwendet. Jumps sind möglich mit `continue` (unterbricht aktuelle Iteration) und `break` (unterbricht gesamten Loop).

```csharp
foreach (ElementType elem in collection)
{ statement }
```

Kriterien für die Collection: Muss `IEnumerable/IEnumerable<T>` implementieren oder einfach die nötigen Members implementiert haben:

Compiler-Output für ein `foreach` Loop:

```csharp
IEnumerator enumerator = list.GetEnumerator();
try
{
	while (enumerator.MoveNext())
	{
		int i = (int)enumerator.Current;
		if (i == 3) continue;			// Ursprüngliche Logik im foreach
		if (i == 5) break;				// ...
		Console.WriteLine(i);			// ...
	}
}
finally
{
	IDisposable disposable = enumerator as IDisposable;
	if (disposable != null)
	disposable.Dispose();
}
```

## IEnumerable / IEnumerator Interfaces

Definiert die Logik, wie in einer Collection "vorwärts gesprungen" wird. Array: Speicher x bytes nach vorne, Linked List: folge der Referenz.

- Hat eine Methode `GetEnumerator()` mit Rückgabewert einer Klasse `e` (Enumerator)
- Enumerator `e` hat eine Methode `MoveNext()` mit Rückgabewert `bool`. Zeigt jeweils auf ein Objekt in der Collection. Zeiger startet vor der Liste, wird beim ersten Aufruf auf das erste Listenelement verschoben.
- `e` hat ein Property `Current`, das jeweils das aktuelle Element zurückgibt. 

Jeder Enumerator ist entweder allgemein (Rückgabetyp `object`, .NET 1.0) oder generisch (Rückgabetyp `T`, leitet jeweils vom nicht-generischen ab).

Wichtig: Rückgabetyp ist nicht Teil der Methodensignatur bezüglich Vererbung/Überladung.

Zwei verschiedene Methoden müssen definiert werden
`object Current {get;}`
`T Current {get;}`

### Mehrfach-Zugriff

Iterator kapselt den State *(wo bin ich)* der aktuellen Traversierung der Liste. Die Collection darf dabei aber nicht verändert werden.
Dies erlaubt unterschiedlichen Threads Arbeit mit derselben Liste gleichzeitig. 

### Implementationsbeispiel

Jeweils zwei Klassen werden benötigt, die einmal IEnumerable und einmal IEnumerator implementieren. IEnumerable wäre jeweils die Collection, Enumerator beinhaltet die Traversierungslogik.

```csharp
// Nicht generisch
class MyList : IEnumerable
{
	public IEnumerator GetEnumerator() { return new MyEnumerator(); }
	class MyEnumerator : IEnumerator // geschachtelt!
	{
		public object Current { get { /* ... */ } }
    	public bool MoveNext() { /* ... */ }
    	public void Reset() { /* ... */ }
	}
}
// Generisch
class MyIntList : IEnumerable<int>
{
	public IEnumerator<int> GetEnumerator() { return new MyEnumerator(); }
	IEnumerator IEnumerable.GetEnumerator() { return GetEnumerator(); }		// Zusätzlich-> Aufruf auf Generisch.
	class MyEnumerator : IEnumerator // geschachtelt!
	{
		public int Current { get { /* ... */ } }
		object IEnumerator.Current { get { /* ... */ } } 					// Zusätzlich
		public bool MoveNext() { /* ... */ }
		public void Reset() { /* ... */ }
	}
}
```

## Keyword yield

Iterator `yield` ersetzt spezifische implementation von IEnumerator. Innerhalb der Methode `public IEnumerator GetEnumerator()` kann jeweils mit einem `yield return` statement der Wert für die Nächste Iteration definiert werden. Typ des Return-Wertes muss mit der Signatur kompatibel sein. 
`yield break` terminiert die Iteration. 

```csharp
class MyIntList				// ...standardmässig wird yield return innerhalb eines Loops verwendet...
{
    public IEnumerator<int> GetEnumerator()
    {
        yield return 1;
        yield return 2;
        yield break;
        yield return 3;		// wird nie aufgerufen..
    }
}
```

Compiler generiert im Hintergrund ein vollständiger Enumerator als State Machine (mit `int __state, int __current`) die jeweils die aktuelle Stelle im GetEnumerator kapselt. Somit kann beim nächsten Aufruf genau dort weitergemacht werden, wo bei der letzten Iteration aufgehört wurde.

### Spezifische Iteratoren

Ermöglicht das festlegen von verschiedenen Iteratoren pro Klasse. Standarditerator wird aufgerufen via `foreach (int elem in list)`. Ein Spezifischer Iterator kann mit einer separaten Methode implementiert werden, der dann nicht mehr ein IEnumerator Objekt zurückgibt, sondern ein IEnumerable, das selber wiederum eine GetEnumerator() Methode besitzt.

```csharp
class MyIntList{
    private int[] _x = new int[10];
    //Standard-Iterator
    public IEnumerator<int> GetEnumerator() 
    {
        for (int i = 0; i < _x.Length; i++)
            yield return _x[i];
    }
    // spezifischer Iterator
    // Kann als Methode (hier) oder auch als Property (Logik innerhalb get {}) umgesetzt werden (keine Parameter)
    public IEnumerable<int> Range(int from, int to)
    {
        for (int i = from; i < to; i++)
            yield return _x[i];
    }
}
foreach (int elem in list.Range(2, 7)) // Aufruf
```

## Deferred Evaluation

Die Iterator-Methode `GetEnumerable()`wird erst ausgeführt, wenn `MoveNext()` aufgerufen wird. Dies passiert im foreach Loop implizit erst dann, wenn effektiv das erste Element abgefragt wird. Somit wird nur so viel Arbeit gemacht, wie effektiv nötig. Dieses Konzept wird bei allen `yield` Aufrufen verwendet. 

## Implementation von Query Operatoren

Werden deferred evaluiert, verwenden Extension Method Syntax und können mit dem `.`-Operator verkettet werden, da sie ihrerseits wieder ein IEnumerable zurückliefern.

```csharp
static class OstExtensions
{
    public static IEnumerable<T> OstWhere<T>(this IEnumerable<T> source, Predicate<T> predicate)
    {
        foreach (T item in source)
        {
            if (predicate(item)) yield return item;
        }
    }
    public static IEnumerable<T> OstOfType<T>(this IEnumerable source)
    {
        foreach (object item in source)
        {
            if (item is T) yield return (T)item;
        }
    }
}
// Anwendung
object[] list = { 4, 3.5, "abc", 3, 6};
IEnumerable<int> res = list
    .OstOfType<int>()
    .OstWhere(k => k % 2 == 0);
foreach (int i in res) { Console.WriteLine(i); } // Filtern der Elemente in 'res' geschieht erst hier
```

Ablauf der Iteratoren im obigen Beispiel

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-deferred-evaluation.png" style="zoom: 40%;" />