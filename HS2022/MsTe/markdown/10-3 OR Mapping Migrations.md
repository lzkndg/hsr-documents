# OR Mapping Advanced, Migrations

Verschiedene Punkte können in relationalen Datenbanken nicht direkt 1:1 umgesetzt werden wie im Code.

## Vererbung

Vorteile: Constraints können über Vererbung einfach abgebildet werden. Bsp: Feld x darf für einen abgeleiteten Typen nicht gesetzt sein, beim anderen ist es zwingend.

Potenzielle Probleme führen dazu, dass diese Varianten oftmals gar nicht angewendet werden:

- Ungenutzte Felder / Speicherplatz
- Zu viele Tabellen
- Inkonsistenzen

Verschiedene Ansätze existieren zur Modellierung von Vererbung in einer Datenbank:

### Table per Hierarchy (EF Core Standard)

Nur über DB Context definierbar. Abgeleitete Klassen werden als separate DbSet-Properties der Basisklasse definiert, oder via Model Builder separat:

```csharp
modelBuilder.Entity<Tablet>
    .HasBaseType<Product>();
```

In der Datenbank wird ein Feld als Diskriminator eingefügt, standardmässig der Name der Klasse. Dies kann bei Refactorings problematisch sein. Deshalb kann der Diskriminator auch übersteuert werden, dann muss jeder instanzierbare Typ definiert sein. Die Basisklasse kann abstrakt definiert werden.

```csharp
modelBuilder.Entity<Product>
    .HasDiscriminator<int>("ProductType")
    .HasValue<Product>(0)
    .HasValue<MobilePhone>(1)
    .HasValue<Tablet>(2);
```

### Table per Type

Vorteil: keine NULL-Felder, Überblick über alle Produkte 
Nachteil: immer Left Joins nötig, wenn alle Informationen benötigt werden.

| Tabelle      | Felder                                 |
| ------------ | -------------------------------------- |
| Products     | Id, Name, Description, Price           |
| MobilePhones | Id (FK), OperatingSystem, SupportsUmts |
| Tablets      | Id (FK), HasKeyboard                   |

### Table per Concrete Type

Vorteil: Einzelne Produkte sind einfach abrufbar
Nachteil: Gesamtüberblick über alle Produkte ist schwieriger

| Tabelle      | Felder                                                      |
| ------------ | ----------------------------------------------------------- |
| Products     | Id, Name, Description, Price                                |
| MobilePhones | Id, Name, Description, Price, OperatingSystem, SupportsUmts |
| Tablets      | Id, Name, Description, Price, HasKeyboard                   |

## Entity / Table Splitting

Mittels Table Splitting können unter anderem mehrere Entitäten auf eine Tabelle gemappt werden. Inheritance (Table per Hierarchy) ist nur eine Variante davon.

### xxxxxxxxxx var currentAssembly = Assembly.GetExecutingAssembly();var classInfo = currentAssembly.GetTypes();​foreach (var classType in classInfo) // Auf jeder Klasse im Assembly{    if (classType.IsDefined(typeof(TableAttribute)))     // Wenn das gesuchte Attribute gesetzt ist    {        Console.WriteLine($"\nKlasse: {classType.Name}");        foreach (var a in classType.CustomAttributes) // für jedes Attribut        {            if (a.AttributeType.ToString() == "_1._2_Attribute.TableAttribute")                 // das gesuchte ausgeben                Console.WriteLine($"Attribute-Argument1: {a.ConstructorArguments[0].ToString()}");        }    }}csharp

Zwei Entitäten auf die gleiche Tabelle mappen. Nur mit FluentAPI möglich. Performance: ermöglicht Abfrage von entweder nur Header Daten oder allen Details

```csharp
public class ShopContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    	modelBuilder.Entity<CustomerDetail>(e => {
    		e.ToTable("Customers");
    		// Map all Detail-Properties
    	});
        modelBuilder.Entity<Customer>(e => {
    		e.ToTable("Customers");
    		// Map Properties
    		e.HasOne(o => o.Details)
    			.WithOne()
    			.HasForeignKey<CustomerDetail>(o => o.Id);
	    });
    }
}
```

### Owned Types

Owned Type hat keinen Primary Key, deshalb keine Convention möglich. Felder des Owned Property in der Tabelle erhalten einen zusammengesetzten Namen aus *OwnedType_Property*: bsp. `Address_Street`

```csharp
public class ShopContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>()
            .OwnsOne(b => b.Address);		// Variante FluentAPI
    }
}

public class Customer {
    public int Id { get; set; }
    public Address Address { get; set; }
}
[Owned] 								// Variante Data Annotations
public class Address { ... }
```

Möglich ist auch, mehrere oder verschachtelte Owned Types zu verwenden.

```csharp
public class Customer {
    public int Id { get; set; }
    public Address BillingAddress { get; set; }
    public Address ShippingAddress { get; set; }
}
/* Ergibt:
BillingAddress_street 	(nvarchar(MAX))
BillingAddress_City		(nvarchar(MAX))
ShippingAddress_Street	(nvarchar(MAX))
ShippingAddress_City	(nvarchar(MAX))
*/
```

## Database Migrations

Naiver Ansatz: Änderungen der Datenbank in einem Change Script festhalten. Probleme dabei:

- keine manuelle Einflussnahme
- schwierig, sobald Daten vorhanden sind
- Umbenennen von Spalten/Tabellen wie DROP / CREATE -> möglicher Datenverlust
- Hinzufügen einer NOT NULL Spalte zwingt auch bestehende Datensätze, NOT NULL einzuhalten

EF Core Ansatz: Modell anpassen, dann eine Migration erstellen. Diese wird als C# Klasse implementiert, die anschliessend reviewed und allenfalls korrigiert werden kann. Jede Migration implementiert die Funktion `up()` und `down()`  und kann als Artefakt in der Versionsverwaltung abgespeichert werden. Deployment wie auch Rollback einfach auszuführen.

### Erstellte Files

`Migrations/<timestamp>\_Migration.cs`							Eigentliche Migration
`Migrations/<timestamp>\_Migration.Designer.cs`			Metadaten für Entity Framework
`Migrations/<contextClassName>ModelSnapshot.cs`			Snapshot / Basis für nächste Migration

### Tabelle

`dbo.__EFMigrationsHistory` zeigt eine Liste der aktuell auf die Datenbank angewendeten Migrations.

### Befehle

| CLI [dotnet ef ...]                                    | Package Manager            | Beschreibung                                                 |
| ------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| `database drop`                                        | `Drop-Database`            | Löscht die gesamte Datenbank                                 |
| `database update`                                      | `Update-Database`          | Setzt den aktuellen Stand der Codebase in der Datenbank um   |
| `database update Init`                                 |                            | Setzt die Datenbank auf den Stand der angegebenen Migration  |
| `migrations add Init`                                  | `Add-Migration <name>`     | Legt eine neue Migration an zum aktuellen Stand **der Codebase** |
| `migrations list`                                      |                            | Zeigt eine Liste von verfügbaren Migrations an               |
| `migrations remove`                                    | `Remove-Migration`         | Löscht die letzte Migration                                  |
| ` migrations script <migration-start> <migration-end>` | `Script-Migration`         | Gibt auf der Konsole ein SQL-Skript aus, das die Änderungen zwischen Migration-Start und Migration-End ausführt. Vorwärts und Rückwärts möglich. |
| `dbcontext info`                                       | `Get-DbContext`            | Liefert Informationen über einen DbContext Typen             |
| `dbcontext list`                                       |                            | Listet verfügbare DbContext Typen                            |
| `dbcontext scaffold`                                   | `Scaffold-DbContext`       | Erstellt einen DbContext und Entity Types für eine bestehende Datenbank |
|                                                        | `Get-Help EntityFramework` | Informationen über Entity Framework Commands                 |

### C# API

Die C# API bietet diverse Operationen um Migrations programmatisch anzuwenden.

```csharp
using(AngProjContext context = new())
{
    DatabaseFacade database = context.Database;
    
    // "Dev" Ansatz (löschen / neu erstellen)
    await database.EnsureDeletedAsync();
    await database.EnsureCreatedAsync();
    
    // Automatische Migration auf neuesten Stand
    await database.MigrateAsync();
    
    // Migrations abfragen
    IEnumerable<string> migrations;
    migrations = database.GetMigrations();
    migrations = database.GetPendingMigrations();
    migrations = database.GetAppliedMigrations();

    // Migration ausführen
    IMigrator m = context.GetService<IMigrator>();
    m.Migrate("<MigrationName>");
}
```

