# LINQ

Language INtegradted Query. Erlaubt es, auf beliebigen Datenquellen (Objekte, XML, Datenbanken etc.) SQL-ähnliche Abfragen zu schreiben.

- Compiler-Geprüft
- Typsicher
- Führt die Lambda-Expression ein
- erlaubt deklarativen Programmierstil mit "Anonymous Types" und "Object Initializers"
- redundante Typinformationen können im Code weggelassen werden

Namespaces: `System.Xml.Linq`, `System.Data.Linq`, etc.

## Architektur

```csharp
List<string> cities = new() { "Rapperswil", "Luzern", "Buchs", "Lausanne" };

IEnumerator<string> enumerator = cities.GetEnumerator();
string elem;
while (enumerator.MoveNext())
{
    elem = enumerator.Current;
    Console.WriteLine(elem);
}
```

LINQ Providers: LINQ To Objects, to Entities (SQL Server), *to SQL (veraltet)*, to XML, to anything..

Array in Query wiedergegeben: SELECT * FROM table;

Query benötigt zum speichern ein Objekt mit Interface `IEnumerable<T>`, um über eine Collection iterieren zu können. LINQ Syntax wird vom Compiler auf Variante Extension Methods umgewandelt.

```csharp
string[] cities = { "Bern", "Basel", "Luzern", "Rapperswil", "Buchs"};

// LINQ Query Expression: Reine Selektion auf ganzes Objekt
IEnumerable<string> q1 =
    from c in cities
    select c;

// Extension Method Variante
IEnumerable<string> l1 = cities.Select(c => c);

// LINQ Query Expression
IEnumerable<string> q2 =
    from c in cities
    where c.StartsWith("B")
    orderby c
    select c;

// Extension Method Variante
IEnumerable<string> l2 = cities
    .Where(c => c.StartsWith("B"))
    .OrderBy(c => c)
    .Select()
```

## Extension Methods

Gesamter Funktionsumfang von LINQ ist beinhaltet in Klasse `System.Linq.Enumerable`. Extension Methods erlauben Ausführung auf einem bestehenden Objekt. Erster Parameter der Definition mit `this` Keyword beschreibt dieses Objekt.

```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static IEnumerable<TSource> Skip<TSource>(
            this IEnumerable<TSource> source, Func<TSource, bool> predicate)
        { ... }
    }
}

// Beispiel Verwendung 
int[] numbers = { 1, 4, 2, 9, 13, 8, 9, 0, -6, 12 };
IEnumerable<int> res = numbers
    .Skip(2)
    .Take(4)
    .Select(k => k * k);
// res = { 4, 81, 256, 64 }
```

### Deferred Evaluation

Query wird erst ausgeführt, wenn auch wirklich eine Abfrage auf dem Enumerator ausgeführt wird mit `MoveNext()`, dies wird mittels `yield return` implementiert. Klappt nur mit `IEnumerable` Typen, andernfalls wird Immediate Evaluation gemacht (Rückgabetyp Liste o.ä).

### Immediate Evaluation

Immer dann, wenn der Rückgabetyp kein `IEnumerable` ist oder eine Aggregierung über alle Elemente gemacht wird. 

Beispiele: `.ToList()`, `.ToArray()`, `.Count()`, `.First()`, `.Sum()`, `.Average()`

## Object Initializers

Erlaubt das Instanzieren und Initialisieren einer Klasse in einem einzigen Statement, auch ohne passenden Konstruktor. Reine Compilertechnologie, wird aufgeschlüsselt in einzelne Zuweisungsstatements.

```csharp
new Test { member = value, ... }; // Default Konstruktor
new Test(value) { member = value, ... }; // Konstruktor mit Argumenten

// Beispiel innerhalb LINQ
class Student { ... }

public void Test()
{
    int[] ids = { 2009001, 2009002, 2009003 };
    IEnumerable<Student> students = ids
        .Select(n => new Student { Id = n });
}
```

## Collection Initializers

Dasselbe gilt für Collections: 

```csharp
// Listen
List<int> list1 = new() { 1, 2, 3, 4 };

// Dictionaries, Hashmap, Hashset
Dictionary<int, string> dict1 = new ()
{
    {1, "a"},
    {2, "b"},
    {3, "c"}
};
// oder
dict1 = new()
{
    [1] = "a",
    [2] = "b",
    [3] = "c"
}
```

![Collection Initializers](C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-collection-initializers-dictionary.png)

Können in Kombination verwendet werden, um komplexe Objektstrukturen aufzubauen.

```csharp
// Macht zwar keinen Sinn, aber hey :)
Dictionary<int, Student> s = new()
{
    { 2009001, new("John") {
        Id = 2009001,
        Subject = "Computing" } },
    { 2009002, new() {
        name = "Ann", Id = 2009002,
        Subject = "Mathematics" } }
};
```

## Keyword "var"

Definiert einen Typ, dessen Namen man nicht kennt. Kommt ausschliesslich in Kombination mit `var` zum Einsatz. Compiler ermittelt, was für ein Typ zugewiesen wird (Type Inference).

```csharp
// Vereinfachung der Lesbarkeit
Dictionary<string, IComparable<object>> dict1 = new Dictionary<string, IComparable<object>>();
var dict1 = new Dictionary<string, IComparable<object>>();

// Erlaubt anonyme Typen
var a = new {Id = 1, Name = "John"};
```

## Anonymous Types

Erzeugung eines strukturierten Wertes, ohne dass der Typ explizit selber angegeben werden muss. Meist für Projektion in LINQ Queries verwendet. Ziel: Zwischenresultate, die nur für eine einzelne Abfrage gelten, müssen mit möglichst wenig Aufwand zwischengespeichert werden.

Syntax: 

`new { "p1" = "v1", "p2" = "v2", ... }`: der Propertyname `p1` - `pn` ist frei wählbar. 

- Alle Properties sind readonly
- Implizite und explizite Angabe der Property Namen können gemischt werden
- Leitet von System.Object ab, Compiler generiert Methoden `equals()`, `GetHashCode()` und `ToString()`
- Kann nur einer Variable vom Typ `var` zugewiesen werden. 
- Name des Typen wird vom Compiler so generiert, dass eine Kollision mit User-Vergebenen Names unmöglich ist (beginnend mit `<>`).

## Query Expressions

LINQ Query Expressions werden in drei Schritten ausgeführt. 

 ````csharp
 // 1. Datenquelle auswählen
 int[] numbers = { 0, 1, 2, 3, 4, 5 }
 // 2. Query erstellen
 var numQuery = from num in numbers
     where (num % 2) == 0
     select num;
 // 3. Query ausführen
 foreach (int num in numQuery)
 {
     Console.Write("{0, 1} ", num);
 }
 ````

| Schlüsselwort                | Bedeutung                                        | Speziell                                           |
| ---------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `from`                       | Definiert Range-Variable und Datenquelle         | Beginnt immer das Query                            |
| `where`                      | Filter                                           |                                                    |
| `orderby field [asc | desc]` | Sortierung                                       |                                                    |
| `select`                     | Projektion                                       | Beendet das Query, wenn kein `group` vorhanden ist |
| `group`                      | Gruppierung in eine Sequenz von Gruppenelementen | Beendet das Query                                  |
| `join`                       | Verknüpfung zweier Datenquellen                  |                                                    |
| `let`                        | Definition von Hilfsvariablen                    |                                                    |

Extension Methoden

| Standard                            | Positional              | Set Operations (bei Joins) |
| ----------------------------------- | ----------------------- | -------------------------- |
| `Select`                            | `First[OrDefault]`      | `Distinct`                 |
| `Where`                             | `Single[OrDefault]`     | `Union`                    |
| `OrderBy[Descending]`               | `ElementAt`             | `Intersection`             |
| `ThenBy[Descending]`                | `Take / Skip`           | `Except`                   |
| `GroupBy`                           | `TakeWhile / SkipWhile` | `Repeat`                   |
| `Join / GroupJoin`                  | `Reverse`               |                            |
| `Count / Sum / Min / Max / Average` |                         |                            |

### Range Variablen

Entstehen durch Klauseln `from, join, into`. Sind Readonly, und nur innerhalb der Expression bis zur nächsten `into` Klausel sichtbar. Typ ist immer `T` bei Datenquelle `IEnumerable<T>`.

### Gruppierung

Bringt eine flache Liste in eine strukturierte Form. Ausgabe `IGrouping<TKey, TElement>` implementiert ein IEnumerable mit zusätzlichem Namen (Key). Dieser repräsentiert das Attribut, welches gruppiert wurde bzw. bei allen Elementen im `IEnumerable` gleich ist. Zugriff auf die Members genau gleich wie bei `IEnumerable` Typen.

```csharp
var q = from s in Students
    group s.Name by s.Subject;
// Ergibt zuerst ein Key Value Pair
{
    {Computing, John},
    {Computing, Sue},
    {Mathematics, Ann}
}

// Danach ein IGrouping<TKey, TElement>
{
    Computing, [ John, Sue ],
    Mathematics, [ Ann ]
}

// Ausgabe mit doppeltem ForEach Loop
foreach (var group in q)
{
    Console.WriteLine(group.Key);
    foreach (var name in group) { Console.WriteLine(" " + name); }
}

// Direkte weiterverwendung mit "into"
var q = from s in Students
    group s.Name by s.Subject into g // ab hier kann nur noch g verwendet werden
    select new 
    {
        Field = g.Key,
        N = g.Count()
    };
```

### Inner Joins

Datensätze, die auf der anderen Tabelle keine Einträge besitzen, fallen weg. 

```csharp
// Explizit - besser für Performance
// on a.X equals b.Y
var q = from s in Students
    join m in Markinggs
    	on s.Id equals m.StudentId
    select s.Name + ", " + m.Course + ", " + m.Mark;

// Implizit - gesamtes Kreuzprodukt wird anschliessend gefiltert
// where a.X == b.Y
var q = from s in Students
    from m in Markings
    where s.Id == m.StudentId
    select s.Name + ", " + m.Course + ", " + m.Mark;
```

### Group Joins

Pro Student wird eine Liste für alle Markings erstellt. Ursprüngliche Tabelle bleibt sichtbar.

```csharp
var q = from s in Students
    join m in Markings
    		on s.Id equals m.StudentId
    	into list						// Enthält die Werte der zweiten Tabelle
    select new
    {
        Name = s.Name,
        marks = list					// Enthält die Werte der zweiten Tabelle
    };
// Ausgabe
foreach (var group in q)
{
    Console.WriteLine(group.Name);
    foreach (var m in group.Marks) { Console.WriteLine(m.Course); }
}
```

### Left Outer Joins

Wie Inner bzw. Group Join, aber Elemente, für die kein Äquivalent gefunden wird, bleiben bestehen. Zwei `from` Statements nötig.

```csharp
var q = from s in Students
    join m in Markings
    		on s.Id equals m.StudentId
    	into list
    from sm in list.DefaultIfEmpty()
    select s.Name + ", " + (sm == null ? "?" : sm.Course + ", " + sm.Mark);
```

### `let` Klausel

Zur Definition von Hilfsvariablen, die berechnet werden können. Lebt für den Rest des Queries.

```csharp
var result = from s in Students
    let year = s.Id / 1000					// Hilfsvariable berechnen
    where year == 2009						// Hilfsvariable filtern
    select s.Name + " " + year.ToString();	  // Hilfsvariable ausgeben
```

### Select Many

Erleichtert das Zusammenfassen verschachtelter Listen, z.B. `List<List<string>>`. Innere Listen von verschiedenen Elementen werden so als eine grosse Liste dargestellt.

```csharp
// Extension Method
var q1 = list.SelectMany(s => s.SomeList);
// Query Syntax
var q2 = from segment in list
    from token in segment
    select token;
```



