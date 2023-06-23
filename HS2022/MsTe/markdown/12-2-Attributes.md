# Attributes

Jede Klasse die von `Attribute` erbt. Eigenes Attribut `AttributeUsage`auf der neuen Klasse definiert die Verwendung. Klassenname hat Suffix -Attribute, in der Verwendung aber nicht mehr. Parameter entweder Positional oder Named.

````csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Field, AllowMultiple=true)]
public class TableAttribute : Attribute
{
    public TableAttribute(string tableName)
    {
        TableName = tableName;
    }
    public string TableName { get; set; }
}
[Table("tblFilme")]
public class Filme { ... }
````

Beispiel CSV-Mapper: Auf jedem Property einer Klasse kann angegeben werden, wie der Name der Spalte im CSV lauten soll und ob auf dem Text eine Transformation (upper- oder lowercase) angewendet werden soll.

```csharp
// Verwendung
public class Address
{
    [CsvName("Name"), Uppercase]
    public string Name { get; set; }
    
    [CsvName("Strasse"), Lowercase]
    public string Street { get; set; }
}
// Definition
public class CsvNameAttribute : Attribute 
{
    public string Name { get; set; }
    public CsvNameAttribute(string name) { Name = name; }
}
public interface IStringFilter { string Filter (string arg); }
public class UppercaseAttribute : Atttribute, IStringFilter
{
    public string Filter (string arg) { return arg.ToUpper(); }
}
```

