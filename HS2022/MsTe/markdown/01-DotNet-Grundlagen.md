# .NET Grundlagen

## .NET Geschichte

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-core.png" alt=".NET Core History" style="zoom: 33%;" />

- __.NET Framework__: nicht mehr weitergeführt. Bis 2019 mit Version 4.8

- __.NET Core__: Multiplatform Implementierung, existiert ab 2016, heisst ab 2020 nur noch .NET und löst Framework ab

- __.NET Standard__: Bietet eine einheitliche Base Class Library (Standard API) für verschiedene Implementationen (Framework, Core, Xamarin, ...). Jede Implementation verwendet eine gewisse .NET Standard Version. 
  - Ziel: Konsistenz und implementation "neutraler" cross-platform Libraries.
  - Kompatibilität mit jeweils verwendeter Core / Framework Version muss gegeben sein.
  - Bei Verwendung in eigenen Libraries: je tiefer die .NET Standard Version, desto einfacher einzubinden, aber desto weniger API Funktionen zur Verfügung.

## .NET Architektur

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-architektur.png" alt=".NET Architektur" style="zoom: 33%;" />

## Common Language Runtime (CLR)

Laufzeitumgebung für .NET Code, analog Java Virtual Machine JVM

- JIT Compiler (Just-In-Time) übersetzt IL Code in Maschinencode
- Speicherverwaltung mit Garbage Collection
- Common Language Specification CLS: ermöglicht cross-language development dank MSIL, sprachübergreifendes Debugging, Type Checking und Code Verification der MSIL (Intermediate Language)
- Common Type System CTS
- Class Loader lädt Klassen-Code zur Laufzeit
- Code Access Security
- Exceptions
- Thread-Verwaltung
- COM-Interoperabilität (Interaktion mit unmanaged Code)

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-clr.png" alt=".NET Common Language Runtime Architektur" style="zoom: 50%;" />

### Microsoft Intermediate Language  (MSIL)

MSIL ist eine vorkompilierte Zwischensprache, und beinhaltet das Common Type System CTS. Sie ist

- Prozessor-unabhängig bzw. Plattformneutral und dadurch portabel
- Assembler-ähnlich
- Sprach-Unabhängig innerhalb der .NET Familie
- Typsicher (Type- und weitere Security-Checks beim Laden des Codes)
- Nachteil Effizienz (wird durch JIT teils wettgemacht, der zur Laufzeit spezifische Maschineninstruktionen nutzen kann)

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-msil.png" alt=".NET Common Language Runtime Architektur" style="zoom: 33%;" />

### Just in Time Compilation (JIT)

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-jit.png" alt=".NET JIT Compiler" style="zoom: 33%;" />

Drei Typen der Kompilierung

- Pre-JIT: Alles vor Ausführung kompiliert
- Normal-JIT: Kompilierung von Methoden bei Aufruf
- Econo-JIT: Wie Normal aber mit Cleanup-Mechanismus

### Common Type System (CTS)

Das einheitliche Typensystem der .NET Umgebung ist in der CLR integriert, somit losgelöst von der einzelnen Programmiersprache. Das Typsystem ist Single-Rooted - das heisst alles ist von `System.Object` abgeleitet, auch alle Primitivtypen.

Reference Types sind alle gängigen Objekte, Klassen etc. (auch Strings), Value Types sind alle Basistypen: _sbyte, byte, short, ushort, int, uint, long, ulong, float, double, decimal, bool, char_

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-datatypes.jpg" alt=".NET Types" style="zoom: 33%;" />

Polymorphismus ist somit für ausnahmslos alle Typen möglich. Methode, die `System.Object` erwartet, kann einen "normalen" Int annehmen.

#### Reference Types

`Class` erzeugt einen eigenen Reference Type. Auf dem Stack wird ein Pointer angelegt, der auf das effektive Objekt im Heap zeigt.

`null` beschreibt immer eine Referenz ohne Adresse / Ziel.

Beinhaltet auch immer eine Referenz auf den Typen. 

```csharp
class PointRef { public int X, Y; }
PointRef a; // erzeugt im Stack eine leere Referenz
a = new PointRef(); // definiert die Referenz im Stack, erzeugt auf dem Heap ein leeres Objekt (Default-Initialisierung von Value Types)
a.X = 12;
a.Y = 24; // jeweiliger Speicherbereich im Heap wird angepasst
```

Zuweisung: Objekt-Referenz wird kopiert. Anpassung an einem Objekt verursacht auch änderungen am anderen Objekt.

```csharp
PointRef a = new PointRef();
a.X = 12;
a.Y = 24;
PointRef b = a;
b.X = 9;
Console.WriteLine(a.X); // Gibt 9 aus
```



#### Value Types

`Struct` erzeugt einen eigenen Value Type. Das Objekt liegt direkt auf dem Stack, **oder inline im Reference- oder Value Type.**

Konstruktor wird vom Compiler automatisch generiert, macht Default-Initialisierung (ausnulle des Speichers). Structs können nicht abgeleitet werden (sealed).

`null` bei Value Types wird immer mit 0 oder false oder der Default-Wert verwendet. Effektiver `null`- Wert nicht möglich.

Vorteile:

- effiziente Speicherausnutzung
- effizienter Zugriff
- keine Garbage Collection nötig, wird nach Ende der Methode abgeräumt

```csharp
struct PointVal { public int X, Y; }
PointVal a; // Im Speicher noch nichts passiert
a = new PointVal(); // Speicher auf dem Stack wird alloziiert und Default-Initialisiert
a.X = 12;
a.Y = 24; // Werte auf dem Stack werden angepasst
```

Zuweisung: gesamter Wert auf dem Stack wird kopiert. 

| Keyword | Aliased Type   | Java    | Wertebereich                    |
| ------- | -------------- | ------- | ------------------------------- |
| sbyte   | System.SByte   | byte    | -128 .. 127                     |
| byte    | System.Byte    |         | 0 .. 255                        |
| short   | System.Int16   | short   | -32'768 .. 32'767               |
| ushort  | System.UInt16  |         | 0 .. 65'535                     |
| int     | System.Int32   | int     | -2'148'483'648 .. 2'147'483'647 |
| uint    | System.UInt32  |         | 0 .. 4'294'967'295              |
| long    | System.Int64   | long    | -2^63 .. 2^63-1                 |
| ulong   | System.UInt64  |         | 0 .. 2^64-1                     |
| char    | System.Char    | float   | +- 1.5E-45 .. +- 3.4E38 (32bit) |
| float   | System.Single  | double  | +- 5E-324 .. +- 1.7E308 (64bit) |
| double  | System.Double  |         | +- 1E-28 .. +- 7.9E28 (128bit)  |
| bool    | System.Boolean | boolean | true, false                     |
| decimal | System.Decimal | char    | Unicode-Zeichen                 |

#### Boxing / Unboxing

Beschreibt das automatische umwandeln von Value Types in Reference Types und zurück. Immer dann, wenn ein Value Type (Struct) als Instanz einem Reference Type zugewiesen wird.

```csharp
System.Int32 i1 = 12; // Value Type
System.Object obj = i1; // Implizites Boxing zum Reference Type
System.Int32 i2 = (System.Int32)obj; // Unboxing muss explizit gecastet werden
```

## Assemblies

Ergebnis der Kompilation, eine Deployment- oder Ausführungseinheit. Vom Typ Executable (`*.exe`) oder Library (`*.dll`). Können dynamisch geladen werden, definiert Typ-Scope. Kleinste versionierbare Einheit. Entspricht einem JAR-File in Java. 

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-assembly.png" alt=".NET JIT Compiler" style="zoom: 33%;" />

#### Module und Metadata

Enthält Code (MSIL) und Metadaten. Metadaten enthalten alle Aspekte des Codes ausser Programmlogik (Definitionen von Klassen, Methoden, Feldern etc.) und können mittels Reflection abgefragt werden. 

- Anwendungen in der CLR: Typsicherheit, Memory Management, JIT Compilation
- Anwendungen der IDE: Object-Browser, IntelliSense
- Tools zur Analyse: IL Disassembler (IL DASM), viele Drittanbietertools
- Erweiterbare Programmsysteme (Generische Ansätze, Dynamic Binding)

## .NET Class Library

- Basisklassen mit System-Funktionen
- ADO.NET / Entity Framework für DB-Zugriff
- ASP.NET (Core) Web-Programmierung
- XML, Dateisystemzugriff
- WPF und Windows Forms (GUI)

## CLI Interface

Teil des .NET Core SDK dient als Basis für high-level Tools wie IDEs. Command Struktur lautet ähnlich wie Git:

`dotnet[.exe] <verb> <argument> --<option> <param>` -- Argument zum Verb (eines), Parameter zur Option (0-n). 

Möglichkeiten (Auszug):

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-cli.png" alt=".NET JIT Compiler" style="zoom: 33%;" />

## Projekte & Referenzen

Projekt-Datei wird seit Version 1.0 als XML verwaltet (`*.csproj` neu seit 2017). Build Engine von Microsoft heisst `MSBuild`, kann direkt oder auch via .NET CLI verwendet werden.

- `<Property*>`: Settings
- `<Item*>`: zu kompilierende Items
- `<Target*>`: Sequenz auszuführender Schritte, TargetFramework definiert die zu kompilierende Ziel-API.

Alte Variante vor 2017 war extrem gross und sperrig. Betreffen nur .NET Framework 4.8 und älter.

Referenzen können im Projekt auf verschiedene Arten geführt werden.

- Vorkompiliertes Assembly: Muss im File System verfügbar sein
- NuGet Package: Externe Dependency (nuget.org)
- Visual Studio Projekt: in selber Solution vorhanden
- .NET Core oder Standard SDK (default, zwingend: `Microsoft.NETCore.App` oder `NETStandard.Library`)

## NuGet Packages

Neuer Standard für Packagin von Applikationen. Teilt die .NET Funktionalität in verschiedene kleinere Packages auf. Erlaubt unterschiedliche Release-Zyklen, erhöht die Kompatibilität durch Kapselung von spezifischen Komponenten, verkleinert die Deployment-Einheiten.

Der Dateityp `*.nupkg` (zip) beinhaltet Assemblies, Manifest mit Infos zu Package Id, Titel und Beschreibung, Versions-Informationen, Dependencies etc.

Ablauf mit Kompilierung (`dotnet build`), Paketierung (`dotnet pack`), Veröffentlichung (`dotnet nuget push`), Konsumation (`dotnet add package`).

Wichtige Packages sind

- `System.Runtime` (Object, String, Array, Action, Func, ...)
- `System.Collections` (Generische Listen wie List<T>, Dictionary<TKey, TValue>, ...)
- `System.Net.Http` (HttpClient, HttpResponseMessage, ...)
- `System.IO.FileSystem` (File, Directory, ...)
- `System.Linq` (Enumerable, ILookup<TKey, TElement>, ...)
- `System.Reflection` (Assembly, TypeInfo, MethodInfo, ...)

## .NET Standard 

Wurde mit .NET Core eingeführt, diente als Brücke zwischen .NET Framework und .NET Core. Motivation: 

- Implementation von "neutralen" Libraries für Cross-Platform Entwicklung. Können von Core und Framework verwendet werden. Base Class Libraries, Beispiel: Zugriff Filesystem / Sockets, Serialisierung von XML, und und und...
- Konsistenz zwischen verschiedenen Frameworks (Fragmentierung minimal halten)

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dot-net-standard.png" alt=".NET JIT Compiler" style="zoom: 33%;" />

_.NET Standard listet minimal zu unterstützende APIs (Klassen und Methoden) auf._

Kompatibilität: Jede .NET Implementation unterstützt eine maximale .NET Standard Version. 

Bei eigenen Libraries: je höher die Version, desto mehr APIs können verwendet werden. Je tiefer, desto einfacher einzubinden.