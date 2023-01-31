# C# Grundlagen

Syntax, neu gegenüber Java

- Referenzparameter
- Objekte am Stack (Structs)
- Blockmatrizen
- Enumerationstypen
- Uniformes Typsystem
- goto
- Systemnahes Programmieren
- Versionierung

"Syntactic Sugar"

- Komponentenunterstützung: Properties, Events
- Delegates
- Indexers
- Operator Overloading
- foreach-Iterator
- Boxing/Unboxing
- Attributes (Annotation)
- ...

### Naming Guidelines

**PascalCase** für so gut wie alles: Namespace, Klassen/Structs, Interface, Enum, Delegates, Methoden, Properties, Events

**camelCase** für Felder, lokale Variablen und Parameter

## Sichtbarkeit

| Attribut          | Wirkung                                                      |
| ----------------- | ------------------------------------------------------------ |
| public            | überall                                                      |
| private           | innerhalb Typ                                                |
| protected         | innerhalb Typ oder Ableitung davon                           |
| internal          | Innerhalb Assembly                                           |
| prtected internal | Innerhalb Typ oder abgeleiteter Klasse oder Assembly         |
| private protected | Innerhalb Typ oder abgeleiteter Klasse, wenn diese im gleichen Assembly ist |

Standards

| Typ       | Standard | Zulässig (nicht nested types) | Standard für Members | Zulässig für Members                                         |
| --------- | -------- | ----------------------------- | -------------------- | ------------------------------------------------------------ |
| class     | internal | public / internal             | private              | public<br />protected<br />internal<br />private<br />protected internal<br />private protected |
| struct    | internal | public / internal             | private              | public<br />internal<br />private                            |
| enum      | internal | public / internal             | public               | --                                                           |
| interface | internal | public / internal             | public               | --                                                           |
| delegate  | internal | public / internal             | --                   | --                                                           |



## Namespaces

Strukturiert den Quellcode, ist hierarchisch aufgbebaut und nicht an physikalische Strukturen gebunden. Mehrere Namespaces im selben File oder Files pro Namespace sind möglich, Unterscheidung von Namespace und Ordnerstruktur auch.

Beinhaltet andere Namespaces, Klassen, Interfaces, Structs, Enums, Delegates

Import eines anderen Namespaces mittels `using System;` -> macht die Verwendung vom `System.*` Präfix unnötig.

Alias Namen sind möglich: `using F = System.Windows.Forms;`

### File-Scoped Namespaces

Erlaubt das entfernen von Klammern, nur ein Namespace pro File erlaubt. 

```csharp
// Klassische Variante: File1.cs
namespace OstDemo
{
    class X { }
}

// File-Scoped NS ab C# 10: File1.cs
namespace OstDemo;
Class X { }
	// Compiler-Fehler bei neuem Namespace im selben File
	namespace OstDemo2 { }
```

### Globale Using Direktive

Meist in einem File `GlobalUsings.cs` , mit `using static Azure.Core` kann das using für das ganze Projekt definiert werden. Verkleinert Boilerplate Code im Header. Alternative im `.csproj`:

```xml
<ItemGroup>
    <Using Include="Azure.Core" />
</ItemGroup>
```

### Implicit Global Usings

Können im `.csproj` aktiviert werden, beinhaltet abhängig vom gewählten SDK verschiedene vordefinierte Usings.

```xml
<Project Sdk="Microsoft.NET.Sdk"> <!-- SDK Version -->
	<PropertyGroup>
		<OutputType>Exe</OutputType>
		<TargetFramework>net6.0</TargetFramework>
		<ImplicitUsings>enable</ImplicitUsings> <!-- Aktivierung -->
    </PropertyGroup>
</Project>
```

## Main-Methode

Einstiegspunkt eines Programms, zwingend für Executable Types. Klassischerweise nur 1x vorhanden, falls mehrere muss in `.csproj`ein Tag `<StartupObject>` definiert werden. Programmlaufzeit ist genau so lange wie die Main-Methode.

Meist in `Program.cs` in der Klasse `Program`. 

Anforderungen:

- Sichtbarkeit ist nicht relevant
- Methode Main muss static sein, Klasse aber nicht
- **Rückgabetypen**: `void`, `int`, `Task` und `Task<int>`, wobei int immer der Program Exit Code ist.
- **Paramtertypen**: keine oder `string[]` für Command-Line Argumente

Zugriff auf Argumente

- über string[] Paramter (meist `string[] args`)
- über statische Methode `System.Environment.GetCommandLineArgs()`
- flexibler CmdLine Parser: NuGet Package `System.CommandLine`

```csharp
class ProgramArgs
{
	static void Main(string[] args)
    {
        for (int i = 0; i < args.Length; i++)
        {
            Console.WriteLine(
                $"Arg {i} = {args[i]}");
        }
    }
}
```

### Top-Level Statements

Erlaubt das Weglassen der Main-Methode als entry point, vereinfacht z.B. Beispiel-Applikationen.

- 1x pro Assembly möglich
- Argumente heissen immer `args`
- Exit Codes sind erlaubt
- `using` werden vorher definiert
- Typen werden nachher definiert

```csharp
using System;
for (int i = 0; i < args.Length; i++)
{
    ConsoleWriter.Write(args, i)
}
class ConsoleWriter
{
    public static void Write(string[] args, int i)
    {
        Console.WriteLine($"Arg {i} = {args[i]}");
    }
}
```

## Enumerationstypen

Liste vordefinierter Konstanten inklusive Wert (default Int32, leitet implizit davon ab). Werte werden von 0 weg aufsteigend vergeben oder können explizit gesetzt werden. Können mit Cast auf `int` verwendet werden. Duplikate sind Okay, werden beim Cast Int zu Enum Type einfach nie verwendet.

```csharp
enum Days { Sunday, Monday, Tuesda, Wednesday, Thursday, Friday, Saturday = 15 }
Days today = Days.Sunday;
int sundayValue = (int)Days.Sunday;

// Enums können auch andere Typen implementieren, bsp. zur Speicheroptimierung. 
enum Days : byte { Sunday, Monday, Tuesday, Wednesday }

// Parsing unter Verwendung der Klasse 'Enum'
Days day1 = (Days)Enum.Parse(typeof(Days), "Monday"); // non-generic

Days day2;
bool success1 = Enum.TryParse("Monday", out day2); // generic
bool success2 = Enum.TryParse("Monday", out Days day3); // generic, ab C# 7.0

// Alle Werte eines Enums ausgeben
foreach(string name in Enum.GetNames(typeof(Days)))
{
	Console.WriteLine(name);
}
```

## Object

Basisklasse aller Typen, `object` ist ein Alias von `System.Object`.

Erlaubt als Parameter dynamische Methoden. Zuweisung von allen Typen kompatibel, da System.Object die gemeinsame Basisklasse aller Typen ist.

Verfügbare Methoden:

```csharp
public Object(); // Konstruktor
public virtual bool Equals (object obj);
public static bool Equals(object objA, object objB);
public virtual int GetHashCode();
public Type GetType();
protected object MemberwiseClone();
public static bool ReferenceEquals(object objA, object objB);
public virtual string ToString();
```

## Arrays

Einfachste Datenstruktur für Listen. Sind zero-Based, letztes Element ist immer length-1. Die Klasse Array ist ein Referenztyp und wird somit immer auf dem Heap angelegt. Alle Werte sind nach Instanzierung initialisiert (false, 0, null, etc.)

Mehrdimensioinales Array rechteckig (Blockmatrix) vs. ausgefranst (jagged).

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-arrays-mehrdim.png" style="zoom:50%;" />

```csharp
// Eindimensional
int[] array1 = new int[5];				 // Deklaration (value type)
object[] arrayA = new object[5]			 // Deklaration (reference type, genau gleich)
int[] array2 = new int[] { 1, 2, 3, 4 };  // Deklaration & Definition
int[] array3 = int[] { 1, 2, 3 }; 		 // vereinfacht ohne new, nur bei deklaration möglich
int[] array4 = { 1, 2, 3 };				 // vereinfacht ohne new und ohne Typ
    
// Mehrdimensional (rechteckig, Blockmatrize)
int[,] multiDim1 = new int[2,3];
int[,] multiDim2 = { {1,2,3},{1,2,3} };
// Mehrdimensional (jagged)
int[][] jaggedArray = new int [6][];
jaggedArray[0] = new int[] { 1, 2, 3, 4 };
```

Länge wird im rechteckigen mehrdimensionalen Array ausmultipliziert, wenn `array1.Length` aufgerufen wird, d.h. die totale Anzahl Speicherplätze ausgegeben (6 im Screenshot oben). Einzelne Dimensionen müssen mit `array1.GetLength(0)` oder `array1.GetLength(1)` abgefragt werden.

Bei jagged Arrays liefert `array2.Length` effektiv die Länge der 0. Dimension (2 im Screenshot oben).

Vorteile von Blockmatrizen:

- Speicherplatz-Effizienz, da weniger Referenzen
- Schneller alloziiert, da ein grosser Block verwendet werden kann
- Schnellere Garbage Collection
- Schnellerer Zugriff nicht wirklich, da allgemein optimiert wird

## Strings

Datenstruktur für Zeichenketten. Referenztyp, verhält sich aber wie ein Value Type. `string` ist Alias für System.String. String selber ist nicht modifizierbar, da interne Optimierung geschieht. Reguläre Verkettung mit + Operator möglich, auch mit Zeilenumbruch. Wertevergleich mit == und Equals wurde überschrieben, um den Inhalt zu vergleichen statt die Referenz. Indexierung möglich. **Nicht** \0 Terminiert. Länge mit Property `Length` abrufbar. Backslash escapen entweder mit @"C:\Temp" oder "C:\\\Temp".

### String Interpolation

x // File 1partial class MyClass{    partial void Test1Initialize();    partial void Test1Cleanup();}​// File 2partial class MyClass{    public void Test2() { }    partial void Test1Initialize() { /*Implementation*/ }}csharp

`string s1 = $"{DateTime.Now}: {"Hello"}";`

`string s2 = $"{DateTime.Now}: {(DateTime.Now.Hour < 18 ? "Hello" : "Good Evening")}";`

### String Interning

Strings werden intern wiederverwendet, d.h. bei gleichem Text wird nur eine neue Referenz auf dasselbe Ziel erstellt. 

`string s2 = String.Copy(string s1);` erstellt eine echte Kopie mit anderer Memory-Adresse.

## Symbole

### Identifiers

Alle Unicode Zeichen erlaubt. Case Sensitive. @ für Verwendung von Schlüsselwörtern als Identifier: `int @while` ist eine gültige Variable.

Unicode Escape Sequenzen sind erlaubt, z.B. \u03c0 anstelle vom Symbol "pi".

### Schlüsselwörter

`abstract, as, base, bool, break, byte, case, catch, char, checked, class, const, continue, decimal, default, delegate, do, double, else, enum, event, explicit, extern, false, finally, fixed, float, for, foreach, goto, if, implicit, in, int, interface, internal, is, lock, long, namespace, new, null, object, operator, out, override, params, private, protected, public, readonly, ref, return, sbyte, sealed, short, sizeof, stackalloc, static, string, struct, switch, this, throw, true, try, typeof, uint, ulong, unchecked, unsafe, ushort, using, virtual, void, volatile, while`

### Kommentare

```csharp
// Single Line
/* Multi Line */
/// Dokumentation auf Typen, Feldern, Methoden, Properties etc.
```

## Primitivtypen

### Ganzzahlen

Bestimmung des Typen:

- ohne Suffix: kleinster Typ aus int | uint | long | ulong
- Suffix u | U: kleinster Typ aus uint | ulong
- Suffix l | L: kleinster Typ aus long | ulong

Syntax: 

- (regulär) digit{digit}{Suffix}: `24L`
- (hex) "0x" hexDigit{hexDigit}{Suffix}: `0xabcdL`
- (bin) "0b" [0|1]{[0|1]}{Suffix}: `0b1001101u`

### Fliesskommazahlen

Bestimmung des Typen

- ohne Suffix: double
- Suffix f | F: float
- Suffix d | D: double
- Suffix m | M: decimal

### Lesbarkeit / Trennzeichen

Underscore erlaubt zur optischen Strukturierung, ausser am Anfang oder Ende einer Zahl. Wird vom Compiler entfernt.

```csharp
object number = 9_876_543_210 // = 9876543210
```

### Zeichenketten

String: "test", Char: 't'

Escape Sequenzen:

- \\' für '
- \\" für "
- \\\ für \
- \0 für 0x000
- \\a für 0x0007 (alert)
- \b für 0x0008 (backspace)
- \f für 0x000c (form feed)
- \\n für 0x000a (new line)
- \\r für 0x000d (carriage return)
- \\t für 0x0009 (horizontal tab)
- \\v für 0x000b (vertical tab)

### Typkompatibilität

![](C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-type-compatibility.png)

### Statements

Leeres Statement: Semikolon ist ein Terminator, kein Separator. 

`;` wie auch `; ; ;` ist jederzeit erlaubt.

Zuweisung, Methodenaufruf, Return Statement mit oder ohne Wert, 

- `if`

- `switch` 

  ​	ganzzahlige, numerische Datentypen, Character und String, Enumeration. 

  ​	`break` beendet den einzelnen `case` - Block (sonst fallthrough), 

  ​	`default` wird ausgeführt wenn nichts zutrifft

- `while` und `do-while`

  ​	`while (true) { }`

  ​	`do {} while (true)`

- `for` und `foreach`

  ​	`for (int i = 1; i <= 3; i++)`

  ​	`foreach (int x in array)`

- Jumps

  ​	`break` - aktuellen Loop beenden

  ​	`continue` - zur nächsten Loop-Iteration

  ​	`goto "myLabel"` -  Sprung zu `myLabel:` (nicht in einen Block, nicht aus dem finally Block eines try-Statements)

  ​	`goto "case"` - innerhalb Switch Statement



