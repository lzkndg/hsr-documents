# Klassen und Structs

## Klasse

Reference Type (wird auf dem Heap abgelegt)

- Basisklassen, Abgeleitete Klassen
- Implementation von Interfaces

Konstruktor mit optionalen Parametern

Initialisierung von Feldern in der Klassendefinition erlaubt.

### Instanzierung

Speicher wird

- alloziiert
- initialisiert

Klassische Initialisierung: `Stack s = new Stack(10);`  
Target-Typed "new"-Expression: `Stack s = new(10);`

## Struct

Soll immer einen einzelnen Wert repräsentieren, speziell wenn dieser aus mehreren Teilen besteht (Vektor, Point, ... ).

Ist immutable, wenn Änderungen nötig sind wird ein neuer Struct erzeugt.

- Kurzlebig oder in andere Objekte eingebettet
- wird initialisiert
- Kann nicht abgeleiten oder als Basisklasse verwendet werden
- Kann Interfaces implementieren

## Felder

- Feld (`int`): initialisierung in deklaration optional,struct felder dürfen nicht initialisiert werden
- Konstante (`cons int`): muss initialisierungswert haben
- readonly-Feld (`readonly int`): in deklaration oder konstruktor initialisiert, wert darf später nicht verändert werden.

## Nested Types

_struct, klasse, interface, enum, delegate:_ Für spezifische Hilfsklassen verwendet.

Zugriff:

- Äussere Klasse sieht __public__ members der inneren
- Innere Klasse sieht __alle__ members der äusseren
- Fremde Klasse sieht __public__ members der inneren

## Statische Klassen

Können nicht instanziert werden. 

Nur statische Members erlaubt

Bsp: Sammlung von Werten oder Funktionalität (`Math.Pi, Math.Cos()`)

## Statische Usings

`using static System.Console` um bsp. Console.WriteLine() zu WriteLine() zu kürzen.

Namenskonflikte müssen manuell aufgelöst werden.

## Statische Methoden

Prozedur/Aktion (ohne Rückgabewert)
Funktion (mit Rückgabewert)

Innerhalb der Klasse ohne Instanz aufrufbar (`Print()`)
Ausserhalb der Klasse eine Instanz nötig (`mc.Print()`)

## Methodenparameter

### Value-Parameter

Kopie des Stack-Inhaltes wird übergeben: Wert (Struct, Int) oder Heap-Referenz (normale Klassen).

__Änderungen an "nicht-Klassen" werden ausserhalb der Methode nicht persistiert!__

### Reference-Parameter

Adresse der Variable wird übergeben. Diese muss initialisiert sein. Nur benötigt für Value Types.

### Out-Parameter

Muss initialisiert werden, kann seit C# 7.0 im Methoden-Header gemacht werden.

`void Init(out int a, out int b) { a = 1; b = 2 }`
`Init(out int a1, out int b1);`

Discarding von out-Parametern möglich mit Underscore:

`bool success = int.TryParse("123a", out _);`

### Params-Array

- Erlaubt beliebig viele Parameter
- Muss am Schluss der Deklaration stehen
- nur einmal erlaubt pro Methode
- Wird in der Methode wie ein normales Array verwendet

```csharp
void Sum(out int sum, params int[] values)
{
    sum = 0;
    foreach (int i in values) sum += i;   
}
```

### Optionale Parameter

Mittels Zuweisung eines Default Values in der Methodendefinition wird ein Parameter optional.

```csharp
private void Sort(
    int[] array,
    int from = 0,
    int to = -1,
    bool ascending = true,
    bool ignoreCase = false
    ) { ... }
```

### Named Parameter

Parameter können mit Namen bezeichnet unsortiert mitgegeben werden, Compiler sortiert anschliessend wieder korrekt:

```csharp
Sort(a, ascending: false, ignoreCase: true)
```

## Überladen von Methoden

Mehrere Methoden mit gleichem Namen: Ünterschiede müssen vorhanden sein in einem von: 

- Anzahl Parameter
- Parametertypen
- Arten (ref/out, spezifisch C#)

Rückgabetyp ist zur Unterscheidung __nicht__ relevant.

## Properties

Reines Compiler-Konstrukt zur Vereinfachung von Get/Set Methoden.
Wie Public Fields, können aber Logik beinhalten beim Lesen und Schreiben.

Standard Property:

```csharp
private _length; // Backing-Field

public int Length
{
    get { return _length; }
    set { _lengt = value; }
}
```

oder als Kurzform:

```csharp
public int LengthAuto { get; set; } // Auto-Property
```

auto-implemented Property:

```csharp
public int LegthInit { get; set; } = 5;
```

Backing-Field wird auf 5 gesetzt.

```csharp
public int LengthInit { get; }
```

Backing-Field wird auf `readonly` gesetzt.

### Objekt-Initialisierung

Properties können direkt initialisiert werden. Dazu eine geschweifte Klammer nach der Klasseninstanzierung verwenden.

```csharp
MyClass mc = new ()
{
    Length = 1,
    Width = 2
}
```

### Init-Only setters

Property muss während der Initialisierung gesetzt werden. Dabei ist Schreibzugriff auf eigene readonly-Felder des Typ möglich.

```csharp
public int LengthInitOnly { get; init; }

MyClass mc = new ()
{
    LengthInitOnly = 42
};
```

## Expression Bodied Members

Bei Methoden mit genau einem Statement. 

```csharp
public class Examples {
    private int _value;

// Constructors / Destructors (C# 7.0)
    public Examples(int v) => _value = v;
    ~Examples() => _value = 0;

// Methods (C# 6.0)
    public int Sum(int x, int y) => x + y;
    public int GetZero() => 0;
    public void Print() => Console.WriteLine("...");

// Properties (C# 6.0)
    public int Zero => 0;
    public int Bla => Sum(Zero, 2);

// Getters/Setters (C# 7.0)
    public int Value {
        get => _value;
        set => _value = value;
    }
}
```

## Indexers

Überladen des Index-Operators, um Klassen mit Array-Verhalten zu erzeugen.

- Read- und/oder Write-Only möglich, weglassen von `get{}`/`set{}`
- Schlüsselwort `this` definiert den Indexer
- Schlüsselwort `value` für Zugriff Wert im Setter
- Indexwert kann überladen werden (mehrere Typen)

```csharp
class MyClass {
    private string[] _arr = new string[10];

    public string this[int index] {
        get { return _arr[index]; }
        set { _arr[index] = value; }
    }

    public int this[string index] {
        get 
        {
            for(int i = 0; i < _arr.Length; i++)
            { 
                if(_arr[i] == search) return i; 
            }
            return -1;
        }
    }
}
```

## Konstruktoren

Gleicher Name wie Klasse, kein Rückgabetyp nötig.


Privater Konstruktor: Kann nur intern verwendet werden, damit wird kein Default-Konstruktor erzeugt.

Aufruf anderer Konstruktoren derselben Klasse mit `this`.
Aufruf des Basisklassen-Konstruktors mit `base`.

### Default-Konstruktor

Konstruktor ohne Parameter.
Kann bei Klasse überschrieben werden, wird ansonsten vom Compiler generiert.
Wird bei Struct immer vom Compiler generiert.

Automatisch generierter Konstruktor setzt alle Felder auf __Standard-Werte__ (`null`, `0`, oder `false`).

Standardwerte können manuell verwendet werden

```csharp
int x = default(int);
int y = default(); // ab C# 7.1 möglich
```

### Statischer Konstruktor

- Zwingend parameterlos
- Keine Sichtbarkeit
- Nur einmal erlaubt
- Wird genau einmal ausgeführt:
  - bei erster Instanzierung des Typen
  - bei erstem Zugriff auf ein statisches Feld

```csharp
Class MyClass() 
{
    static MyClass() 
    {
        // ...
    }
}

```

### Destruktoren

Finalizer bei Java
Bei Klassen erlaubt, bei Structs verboten (keine Möglichkeit zum Ausführen von Logik beim Abräumen)

```csharp
class MyClass()
{
    ~MyClass()
    {
        // Freigabe von Ressourcen ... 
    }
}
```

## Initialisierungs-Reihenfolge

__Klasse Sub__: Statische Felder -> Statischer Konstruktor -> Felder  
--> __Klasse Base__: Statische Felder -> Statischer Konstruktor -> Felder  
--> --> __Klasse Object__: Statische Felder -> Statischer Konstruktor -> Felder  
--> __Klasse Base__: Konstruktor  
__Klasse Sub__: Konstruktor

### Konstruktoren in Ober- oder Unterklasse

Impliziter und expliziter Aufruf des Basisklassen-Konstruktor

```csharp
class Sub : Base
{
    public Sub(int x)
        :base(x)
        {
            // ...           
        }
}
```

## Operator Overloading

Methode muss `static` sein, erzeugt eine eigene Instanz.

```csharp
public static Point operator + (Point a, Point b)
{
    return new (a._x + b._x, a._y + b._y);
}
```

## Partielle Klassen

_Klassen, Structs, Interfaces_

Definition von einem Typen in mehreren Files möglich.

Zweck: 

- Arbeiten mit Generatoren
- Aufsplitten grosser Dateien
- Arbeiten mit schlechter Codequalität

Selbe Sichtbarkeit, Keyword `partial` zwingend bei allen Klassenteilen
Implementation von Interfaces, sealed kann auch über alle Teile verwendet werden.

## Partielle Methoden

Ermöglicht Benutzer-Definierte _Hooks_ in generiertem Code.

Funktioniert in Klassen / Structs (muss ebenso `partial` sein).

Kann, muss aber nicht implementiert werden:

```csharp
// File 1
partial class MyClass
{
    partial void Test1Initialize();
    partial void Test1Cleanup();
}

// File 2
partial class MyClass
{
    public void Test2() { }
    partial void Test1Initialize() { /*Implementation*/ }
}
```
