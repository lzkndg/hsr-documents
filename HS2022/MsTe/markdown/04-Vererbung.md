# Vererbung

## Grundlagen

Regeln:

- Nur eine Basisklasse, beliebig viele Interfaces
- Structs können nicht erweitert werden oder erben, aber Interfaces implementieren
- Alle Klassen sind direkt oder indirekt von `System.Object` abgeleitet
- Structs sind über Boxing mit `System.Object` kompatibel

Unterschieden wird zwischen dem **statischen Typ** der Variable und dem **dynamischen Typ** des Objekts auf dem Heap.

Zuweisung ist erlabut wenn statisch = dynamisch oder statisch Basisklasse von dynamisch, nicht erlaubt wenn statisch Subklasse von dynamisch, oder beide nicht in derselben Vererbungshierarchie sind.

### Typprüfungen

- `<var> is <Class>` liefert true oder false zurück, wenn der Typ identisch oder eine Subklasse davon ist. 
- Explizite Type Casts mit `(<class>)<var>` weisen den Compiler an, einen Typ in einen anderen umzuwandeln
- Umwandlung mit `<var> as <Class>` bewirkt den Output `obj is T ? (T)obj : (T)null`
- Prüfen mit casted als `if (a is SubSub something) { .. }` => `SubSub something = default; if (a is SubSub) { something = (SubSub) a}`. Arbeitet also mit  `default` statt `null` wie "as"-Operator.

## Klassen

### Methoden

Subklasse kann Members der Basisklasse überschreiben. Schlüsselwort `virtual` macht Basis-Methode überschreibbar, `override` um in Subklasse effektiv zu überschreiben. Wichtig: beide müssen immer in Kombination angegeben werden, sonst wird nichts überschrieben. Signatur der Methoden muss exakt gleich sein.

Kombination `virtual` nicht möglich mit `static, private, override`, auch nicht mit `abstract` da dies `virtual` impliziert.

### Dynamic Binding

Erlaubt das Aufrufen einer Methode vom dynamischen Typ, auch wenn der statische Typ ein anderer ist. FALLS die Methode im statischen Typ `virtual` markiert ist, suche Vererbungshierarchie von oben nach unten nach konkretester Methode mit Schlüsselwort `override`.

Methoden überdecken: wenn gewünscht, Keyword `new` verwenden.

Ablauf der Methodenwahl: Für jeden Type der Hierarchie zwischen static type und dynamic type: wenn override Method vorhanden, wähle diese. sobald eine non-override methode (new) gefunden wird, breche ab und verwende die zuletzt gefundene Methode.

**Achtung**: Gecastet wird immer der Static Type!

### Abstrakte Klassen

Mischung aus Klasse und Interface. Kann nicht direkt instanziert werden. Kann beliebig viele Interfaces implementieren, wenn alle Members deklariert sind. Abgeleitete Klassen müssen alle abstrakten Members implementieren, Abstrakte Members können nur innerhalb abstrakter Klassen deklariert werden.

### Abstrakte Methoden

Kein Anweisungsteil, sondern direkt mit ; abgeschlossen: `public abstract void Add(object x);`.  Implizit virtual, dürfen nicht static oder virtual deklariert werden.

### Abstrakte Properties

Kein Anweisungteil, get und set mit Semikolon abgeschlossen. Implizit virtual, dürfen nicht static oder virtual deklariert werden. `get/set` Kombination muss bei Implementation identisch sein.

### Versiegelte Klassen

Keyword `sealed` verhindert das Ableiten einer Klasse, analog Java `final`. Aspekte: Sicherheit (kein versehentliches Erweitern), Performance (Methoden können statisch gebunden werden). Verwendung bei Klassen und deren Methoden, Properties, Indexern und Events.

### Versiegelte Members

Verhindert das Überschreiben eines bestimmten Klassenmembers. Kann nicht in einer abstrakten Klasse vorkommen. Ableitung einer Abstrakten Klasse kann Members `public override sealed` definieren, um weitere Ableitungen zu verhindern. `public new virtual` desselben Members ist in abgeleiteten Klassen aber wiederum möglich.

## Interfaces

Kann nicht direkt instanziert werden, keine Sichtbarkeit auf Members. Kann andere Interfaces erweitern. Members sind implizit `abstract virtual`. Dürfen nicht `static` sein oder ausprogrammiert werden. Name beginnt mit grossem `I`.

Erlaubte Inhalte: Methoden, Properties, Indexers, Events

### Implementieren

- Klasse kann beliebig viele Interfaces implementieren
- Alle Interface-Members müssen auf der Klasse vorhanden sein, entweder direkt implementiert oder von anderer Basisklasse geerbt
- `override` ist nicht nötig, ausser Basisklasse definiert den gleichen Member
- Kombination mit `virtual` und `abstract` ist erlaubt
- Implementierte Interface-Members müssen `public` und nicht `static` sein.

### Verwenden

```csharp
interface ISequence
{
	void Add (object x);
	string Name { get; }
	object this[int i] { get; set; }
	event EventHandler OnAdd;
}
class List : ISequence
{
	public void add(Object x) { /* Implementierung */}
    public string Name { get { /* Implementierung */ } }
    public object this[int i] { get { /* Implementierung */ } set { /* Implementierung */ } }
    public event EventHandler OnAdd;
}
// Zuweisung/Verwendugng
List list1 = new List();
list1.Add("Hello");
ISequence list2 = new List();
list2.Add("Hello");
// Typumwandlung
list1 = (List)list2;
list1 = list2 as List;
list2 = list1;
// Typprüfung
if (list1 is ISequence) { /* ... */ }
```

## Naming Clashes

Zwei Interfaces könnten dieselben Member definieren, was zu einer Kollision führt. 

Variante 1: Selbe Signatur, selber Rückgabetyp, und Logik ist identisch. Dann kann die Methode ganz normal implementiert werden, ohne weitere Angaben.

Variante 2: Selbe Signatur, selber Rückgabetyp, Logisch ist unterschiedlich. Dann müssen die Methoden zur Implementierung explizit benannt werden. Bsp. `void ISequence.Add(object x) { ... } void IShoppingCart.Add(object x) { ... }`

Variante 3: Selbe Signatur, selber Rückgabetyp, Logisch ist unterschiedlich aber ein Default-Verhalten ist definierbar. Dann kann eine Methode 'normal' und eine Methode explizit implementiert werden. Bsp. `void Add(object x) { ... } void IShoppingCart.Add(object x) { ... }`

## Fragile Base Class Problem

Das Markieren von `virtual` und `override` verhindert, dass bei nachträglich hinzugefügten Methoden in einer Basisklasse ein Konflikt mit allenfalls schon vorhandenen Methoden in der Subklasse mit demselben Namen entstehen. Schlimmstenfalls wirft der Compiler eine Warnung, wenn der Code neu kompiliert wird. In Java beispielsweise führt das ohne Warnung zu anderem Verhalten.