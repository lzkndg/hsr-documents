# Extension Methods

Erlauben das erweitern von bestehendne Klassen aus Anwendersicht. Die Signatur der Klasse wird jedoch nicht verändert. Zur Deklaration:

- Methode muss in statischer Klasse sein
- Methode muss statisch sein
- Erster Parameter definiert, auf welcher Klasse die Methode verfügbar ist und hat ein `this` Schlüsselwort voranstehend.

```csharp
public static class ExtensionMethods
{
    static string ToStringSafe(this object obj)
    {
        return obj == null ? string.Empty : obj.ToString();
    }
}
```

Der Compiler wandelt den Aufruf von `1.ToStringSafe()` in `ExtensionMethods.ToStringSafe(1)` um. Typsicherheit ist damit gewährleistet.

Regeln:

- Erlaubt auf Klassen, Structs, Interfaces, Delegates, Enumerators und Arrays
- Kein Zugriff auf interne Members
- Methode ist nur sichtbar, wenn der Namespace der beinhaltenden Klasse importiert wurde
- Bei Konflikt mit einer Methode der Zieklasse verliert die Extension Method immer

