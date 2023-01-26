# Generics

## Grundlagen

Typsichere Datenstrukturen ohne Festlegung auf einen fixen Typen. Anstelle dessen wird `<T>` verwendet. Bei der Verwendung wird T jeweils als spezifischer Typ deklariert. Hat gegenüber der Verwendung von `object` den Vorteil der Typsicherheit. Weiter hohe Wiederverwendbarkeit und Performance.

Hauptanwendungsfall: Collections. Ohne Generics müsste eine Variante pro Collection für jeden einzelnen Typ implementiert werden. Oder bei Verwendung von Object wären ständig Boxing/Typumwandlungen nötig.

Generisch sein kann *Klasse / Struct / Interface / Delegate / Event*. Sowie einzelne Methoden, auch wenn die Klasse nicht generisch ist.

Deklaration von T immer nach dem Namen des Elements.

Standard-Namensgebung bei einem generischen Parameter: "T", bei mehreren: "T1", "T2" etc. oder "TElement", "TKey" etc.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-generics-laufzeiteffizienz.png" style="zoom:45%;" />

## Type Constraints

Verschiedene Varianten von Constraints sind auf generischen Typen möglich. Angabe jeweils direkt nach bzw. auf der nächsten Zeile nach der Definition. Compiler prüft den Constraint. Kombinationen von verschiedenen Constraints kommagetrennt, pro T-Parameter eine where-Klausel mit Leerzeichen getrennt.

```csharp
class OrderedBuffer<TElement, TPriority> 
    where TPriority : IComparable, SomeConstraint
    where TElement : SomeConstraint
{ ... }
```

| Constraint                  | Bedeutet                                                     |
| --------------------------- | ------------------------------------------------------------ |
| `where T : struct`          | T muss ein Value Type sein, heisst liegt auf dem Stack oder inline im anderen Objekt. |
| `where T : class`           | T muss ein Reference Type sein, also Klassen oder auch Interfaces, Delegates. Liegt auf dem Heap. |
| `where T : new()`           | T muss einen parameterlosen "public" Konstruktor haben, kann also instanziert werden. *Muss immer zuletzt angeführt werden, wenn mit anderen Constraints kombiniert.* |
| `where T : "ClassName"`     | T muss von der angegebenen Klasse ableiten, bietet also alle Members. |
| `where T : "InterfaceName"` | T muss das Interface implementieren                          |
| *`where T : TOther`*        | *T muss identisch sein wie oder ableiten von TOther -> bei Extension Methods verwendet* |
| `where T : class?`          | T muss ein Nullable Reference Type sein. (Klassen, Interfaces, Delegates) |
| `where T : not null`        | T muss ein Non-Nullable Value Type oder Reference Type sein  |

Motivation: möglichst wenige Einschränkung, nur genug um die gewünschte Logik zu implementieren.

## Vererbung

Verschiedene Varianten möglich:

```csharp
class MyList<T> : List { }		// Erben von normalen Klassen
class MyList<T> : List<T> { }	// Weitergabe des Typparameters an generische Basisklasse
class MyIntList : List<int> { }	// Konkretisierte generische Basisklasse
class MyIntKeyDict<T> : Dictionary<int, T> { }	// Mischform
// Achtung: Typparameter werden nicht vererbt
class MyList : List<T> { } // Compilerfehler!
```

### Zuweisungen

Zuweisung einer Instanz eines generischen Typen an nicht-generische Basistypen ist immer möglich. Wenn die Basisklasse schon generisch ist, müsseen Typparameter auf jeden Fall kompatibel sein.

```csharp
class MyList<T> { }
class MyList2<T> : MyList<T> { }
class myDict<TKey, TValue> : MyList<TKey> { }
class Examples
{
    public void Test()
    {
        MyList<int> l1 = new MyList2<int>();
        MyList<int> l2 = new MyDict<int, float>();
        // Compilerfehler
        MyList<int> l3 = new MyList<float>();
        MyList<object> l4 = new MyList<float>(); // wg. Co-/Contravarianz, einfügen von Objects 
    }
}
```

### Methoden überschreiben

```csharp
class Buffer<T> { public virtual void Put(T x) } // Basis
// Konkretisierte Basisklasse
class MyIntBuffer : Buffer<int>
{ public overrride void Put(int x) { ... } }
// Generische Vererbung
class MyBuffer<T> : Buffer <T>
{ public override void Put(T x) { ... } }
```

### Typprüfungen und Type Casts

Können beide wie mit normalen Typen durchgeführt werden. Prüfung identisch wie für normale Typen mit `is`-Operator, Cast mit Klammer oder `as`-Operator. Reflection unterstützt abfrage von konkretisierten und nicht konkretisierten Typparametern:
`Type t = typeof(Buffer<int>)`.

### Generic Type Inference

Bei Methoden kann der Typparameter weggelassen werden, wenn T ein formaler Parameter ist. Damit wird der Typ vom Compiler ermittelt. Rückgabewert zählt nicht.
Methode `public void Print<T>(T t) { }` kann direkt aufgerufen werden als `Print(12)`. 
Methode `public T Get<T>() { }` muss immer der Typ mitgegeben werden mit `Get<int>()`.

### Konkretisierung auf CLR Ebene

Bei Value Types wird für jeden neuen Typen eine eigene Klasse mit konkreter Implementierung erstellt -> Performance. Für Reference Types wird einmal eine Klasse mit Typ `<object>` erzeugt und für jeden anderen Typen wiederverwendet.

## Generische Delegates

Auch Delegates unterstützen generische Typen. 

```csharp
public delegate void Action<T> (T i);
public class MyClass
{
    public static void PrintValues<T>(T i) { Console.WriteLine("Value {0}", i); }
}
public class FunctionParameterTest
{
    static void ForAll<T>(T[] array, Action<T> a)
    {
        Console.WriteLine("ForAll called...");
        if (a == null) { return; }
        foreach (T t in array) { a(t); }
    }
}
```

In .NET vorimplementierte Delegates (in/out => Ko-/Kontravarianz)

- `public delegate void Action<in T1, in T2>(T obj1[, T obj2])`
- `public delegate TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2)`
- `public delegate bool Predicate<in T1, in T2>(T1 obj1, T2 obj2)` (spezialfall vom Func Delegate. Rückgabetyp fixiert auf Boolean)

Event Handler Delegate: 

`public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e)`

## Generische Collections

| Namespace `System.Collections.Generic`   | Namespace `System.Collections` |
| ---------------------------------------- | ------------------------------ |
| `List<T> / IList<T>`                     | ArrayLIist / IList             |
| `SortedList<TKey, TValue>`               | SortedList                     |
| `Dictionary / IDictionary<TKey, TValue>` | Hashtable / IDictionary        |
| `SortedDictionary<TKey, TValue>`         | SortedList                     |
| `LinkedList<T>`                          | -                              |
| `Stack<T>`                               | Stack                          |
| `Queue<T>`                               | Queue                          |
| `IEnumerable<T> / IEnumerator<T>`        | IEnumerable / IEnumerator      |
| `ICollection<T>`                         | ICollection                    |

​		![](C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-collections-interfaces.png)

## Ko- und Kontravarianz

Nicht prüfungsrelevant.