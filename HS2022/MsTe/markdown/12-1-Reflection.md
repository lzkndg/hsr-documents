# Reflection

## Anwendung: Dynamische  Programmierung

- Metadaten darstellen: Intellisense, Object Browser, etc0.

- Type Discovery: Suchen und Instanzieren von Typen, Zugriff auf Dynamische Datenstrukturen (bsp. JavaScript-Objekte)
- Late Binding von Methoden oder Properties: Suche nach Methoden via String, Ausführen "on the fly"
- *Reflection Emit: Erstellen von Typen inkl. Members zur Laufzeit. Nicht in .NET Core portiert worden.*

## Basics

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dotnet-gettype.png" style="zoom: 50%;" />

Alle Typen in C# sind selbstbeschreibend. Erfolgt über eine Instanz der Klasse `System.Type`, die z.B. die Klasse String beschreibt. Kann programmatisch abgefragt werden nach Visiblity, Properties, alle weiteren möglichen Schlüsselwörter. Meist wird mit der Ableitung `System.RuntimeType` gearbeitet.

Vererbungshierarchie: Befindet sich im Assembly *mscorlib*, Namespace System.Reflection wenn nicht anders angegeben. MemberInfo ist abstrakte Basisklasse die gemeinsame Informationen der abgeleiteten Typen zusammenfasst. Genauso MethodBase.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\dotnet-object-hierarchy.png" style="zoom:50%;" />

System.Type beschreibt alle möglichen Eigenschaften eines Typen, unter Anderem:
- FullName
- IsAbstract
- Klasse oder Struct
- GetFields: Liste von Feldern / Klassenvariablen

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\csharp-reflection-aufrufpfade.png" style="zoom: 50%;" />

Objekttyp BindingFlags ist als Bitmask implementiert und können mittels Oder-Verknüpfung kombiniert werden. Dienen zum Filtern von verschiedenen Infos.

### Type Discovery

BindingFlags: Public/NonPublic, Static/Instance, DeclaredOnly. Defaultmässig werden nur Public Information ausgegeben.

```csharp
Type[] t01 = assembly.GetTypes();
foreach (Type type in t01)
{
    Console.WriteLine(type);
    // Alle Arten von Members
    MemberInfo[] mInfos = type.GetMembers();	
    // dynamische Abfrage von Members
    BindingFlags bf = BindingFlags.Public | BindingFlags.Static;
    MemberInfo[] mInfos = type.FindMembers(MemberTypes.Method, bf, Type.FilterName, "Get*");
    // Ausgabe
    foreach (memberInfo mi in mInfos)
    	{ Console.WriteLine("\t{0}\t{1}", mi.MemberType, mi); }
}
// Ausgabe: ...
// System.Int32
// 		Method Int32 CompareTo(System.Object)
//		Method Int32 CompareTo(Int32)
//		Method Boolean Equals(System.Object)
//		Method Boolean Equals(Int32)
//		Method Int32 GetHashCode()
//		Method System.String ToString()
//		...
// ...
```

### FieldInfo 

Beschreibt ein Feld auf seiner Klasse mit Name, Typ, Sichtbarkeit etc. Ermöglicht direkten Lese- / Schreibzugriff an API vorbei. Casts zum jeweiligen Typen nötig.

```csharp
Fieldinfo[] fiAll = type.GetFields( BindingFlags.NonPublic );
int value = (int)fi.GetValue(object obj);	// Instanz des Objekts
fi.SetValue(object obj, object value);		// Instanz des Objekts und neuer Wert
```

### PropertyInfo 

(Lese- / Schreibzugriff prüfen mit `CanRead`/`CanWrite`)

```csharp
PropertyInfo[] piAll = type.GetProperties();	// Alle Properties
PropertyInfo pi = type.GetProperty("SomeProperty"); // Spezifisches Property
// Permissions prüfen mit pi.CanRead und pi.CanWrite
```

### MethodInfo / ConstructorInfo

Methoden können aufgelistet, nach Namen oder auch nach Parameter-Typen gefunden werden: 

```csharp
MethodInfo[] miAll = type.GetMethods();
MethodInfo mi = type.GetMethod("Increment");
// Abfrage mit Parametern
Type[] paramTypes = {typeof(int), typeof(string)};
MethodInfo miAbs = type.GetMethods("name", paramTypes);
// Aufruf mit Parametern
object[] @params = { -1, "test" };
miAbs.Invoke(type, @params);

// Konstruktoren
ConstructorInfo[] ciAll = type.GetConstructors();
ConstructorInfo ci01 = type.GetConstructor(new[] { typeof(int) }); // Type Array paramTypes on the fly definiert
// Aufruf
Counter c01 = (Counter)ci01.Invoke(new object[] { 12 }); // Parameter array on the fly definiert

// Konstruktor ausführen via Activator
Counter c02 = (Counter)Activator.CreateInstance(
    typeof(Counter), 12 // mehr parameter mit komma getrennt..
	); 
// oder bei public default constructors
Counter c03 = Activator.CreateInstance<Counter>();
```

### BindingFlags

`Public / NonPublic`, `Static / Instance`, `DeclaredOnly` (nicht vererbt)

Deaktivieren von Reflection-Zugriffen => Code Access Security CAS

## Weitere Beispiele

### Assemblies

```csharp
FileInfo fi = new("../../../Tiere.dll");				    // Laden von dll-File via Pfad
Assembly tiere = Assembly.LoadFile(fi.FullName);
Assembly specificAssem = Assembly.Load("mscorlib");		     // Laden von dll via Name
Assembly assemFromType = t.Assembly;					   // Assembly eines spezifischen Typs
Assembly currentAssem = Assembly.GetExecutingAssembly();	// Aktuell ausgeführtes Assembly
```

### Objekte Instanzieren und Methoden ausführen

```csharp
FileInfo fi = new("../../../Tiere.dll");
Assembly tiere = Assembly.LoadFile(fi.FullName);

Type katzeTyp = tiere.GetType("Tiere.Katze");
ConstructorInfo[] ciAll = katzeTyp.GetConstructors();

ConstructorInfo c01 = ciAll[0];
Console.WriteLine(c01);		 // Void .ctor(System.String)

object mina = c01.Invoke(new[] {"Mina"});xi

MethodInfo[] miAll = katzeTyp.GetMethods();
Type[] paramTypes = { };
MethodInfo miMausFangen = katzeTyp.GetMethod("MausFangen", paramTypes);
        
for (int i = 0; i < 5; i++)
{
	miMausFangen.Invoke(mina, null);
    Console.WriteLine(mina);			// Die Katze Mina hat erst eine Maus gefangen. ...
}
```

### Attributes abfragen

`ICustomAttributeProvider` wird instanziert von Assembly/Module, Type, MemberInfo, ParameterInfo

```csharp
var currentAssembly = Assembly.GetExecutingAssembly();
var classInfo = currentAssembly.GetTypes();

foreach (var classType in classInfo) // Auf jeder Klasse im Assembly
{
    if (classType.IsDefined(typeof(TableAttribute))) 
    // Wenn das gesuchte Attribute gesetzt ist
    {
        Console.WriteLine($"\nKlasse: {classType.Name}");
        foreach (var a in classType.CustomAttributes) // für jedes Attribut
        {
            if (a.AttributeType.ToString() == "_1._2_Attribute.TableAttribute") 
                // das gesuchte ausgeben
                Console.WriteLine($"Attribute-Argument1: {a.ConstructorArguments[0].ToString()}");
        }
    }
}
```
