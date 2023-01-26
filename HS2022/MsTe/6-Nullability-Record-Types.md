# Nullable, Record Types

## Default Operator

`default(T) / default` Liefert einen Default-Wert für jeden angegebenen Typen. Bei Reference Types: `null`, bei Value Types `0`, `\0`  oder `false`.

## Nullable

`Null` = ein Null Pointer, also eine Referenz die (noch) keine Zuweisung hat. 

### Value Types

Ein `int` kann normalerweise nur 0 sein, nicht `null`. Mit dem `?` Operator kann jeder Value Type auch `null`sein. Meist bei Datenverbindungen benötigt, da dort jeder Wert auch "nicht vorhanden" sein kann.

`System.Nullable<T>.HasValue` um auf Null zu prüfen. Einfacher Wrapper, der den Null-Zustand für Value Types abbildet. Zugriff auf die Variable, wenn der Zustand Null ist, wirft eine `InvalidOperationException`.

```csharp
public struct Nullable<T> where T : struct
{
    public Nullable (T value);
    public bool HasValue { get; }
    public T Value { get; }
}
int? x = 123; 		// entspricht einem System.Nullable<int>
int x1 = x.HasValue ? x.Value : default; 	// Umwandlung zurück zum Int in 3 Varianten
int x2 = x.GetValueOrDefault();
int x3 = x.GetValueOrDefault(-1);
int i = 123 ; int? x = i; int j = (int)x;	// Mögliche InvalidOperationException.
```

### Reference Types

Nullable Reference Type: 	Muss enabled oder disabled werden. Geschieht im `.csproj` File oder im Code mit `#nullable enable` oder `#nullable disable`, bzw. `#nullable restore` für Projekt-Default.

```xml
<PropertyGroup>
<…/>
<Nullable>[disable|enable]</Nullable>
<…/>
</PropertyGroup>
```

**Wenn Nullable disabled**: Jeder Reference Type kann jederzeit auch `null` sein.
**Wenn Nullable enabled**: Wenn ein Reference Type `null` sein kann, muss er mit dem `?` Operator versehen werden.

```csharp
string? x = null;
string y = null; // Compiler-Fehler.
```

### Null-Checks

 `x is null` / `is not null` prüft in jedem Fall auf eine Null-Referenz. Bei Value Types wird `HasValue` geprüft, bei Reference Types gilt `object.ReferenceEquals(x, null);`

`x == null` wahrscheinlich auch, der == Operator könnte aber überschrieben worden sein.

### Weitere Operatoren

```csharp
int? x = null; 
int i = x ?? -1;	// null-coalescing operator: entweder Wert von x oder -1 wenn null

int? x = null;
i ??= -1;			// null-coalescing assignment-operator: wenn i = null dann -1 zuweisen

Action a = null;
a?.Invoke();		// null-conditional operator: rechten Teil ausführen, wenn nicht null

int? x = null;
int y = x!			// null-forgiving operator: übersteuert den Compiler (wenn du denkst, du weisst es besser)
```

## Record Types

Reines Compiler-Feature zur Darstellung von Klassen, die Daten repräsentieren. Ziele:

- Reduziert den Boilerplate Code
- Vereinfacht die Arbeit mit Nullable Reference Types
- Record Types sollen immutable sein

```csharp
// Beispiel Positional Syntax, kompakteste Schreibweise. Definition von Werten in Klammern nach Header
public record [class|struct] Person(
    int id, 
    string name
);
// Beispiel "normale" Syntax, mehr manuelle Implementation. Automatisch: ToString und Equality wird generiert
public record Person 
{
    public Person() : this(0, "") { }
    public Person(int id, string name) { Id = id; Name = name; }
    
    public int Id { get; init; }		// Init Only Properties
    public string Name { get; init; }
}
// Mischung von beiden Varianten ist möglich. 
```

Der Compiler generiert dazu bestimmte Members:

- Konstruktor
- Properties (init-only)
- Value equality
- Darstellung mit ToString
- Vererbung (z.B. Equality) wird berücksichtigt

### Weitere Features

Vererbung: Basisklasse entweder System.Object oder auch ein Record.

```csharp
public abstract record Person(int Id);
public record SpecialPerson(int Id, string Name) : Person(Id);
// Legt das Id Property in der SpecialPerson Klasse nicht neu an.
```

Value Equality: Vergleich mit `==` oder `.Equals()` vergleicht jedes einzelne Property, involviert auch Properties allfälliger Basisklassen. Nur `ReferenceEquals()` vergleicht wirklich die Memory Adresse.

Nondestructive Mutation: Vereinfacht das Erzeugen leicht modifizierter Kopien. 

```csharp
Person p1 = new(0, "Ava");
Person p2 = p1 with { Id = 1 }; // Gleicher Name, andere Id
```













Null forgiving Operator `"!"`:
wenn du denkst, du weisst es besser 

extrem schlechtes Beispiel.. `null!`

.. dann lüg ich ihn an, und der Compiler glaubt mir das. 