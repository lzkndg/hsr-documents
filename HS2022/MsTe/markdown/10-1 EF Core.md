# Entity Framework Core

Ein providerbasiertes OR Mapping Framework. Neue Provider sind als Pakete über NuGet verfügbar und müssen nicht per se von Microsoft sein. Gängige Provider sind MS SQL Server, SQLite, PostgreSQL, In-Memory Provider.

Basis-Funktionalitäten von EF Core beinhalten:

- Mapping von Entitäten
- CRUD-Operationen
- Key-Generierung
- Caching (Performance)
- Change Tracking (Logs)
- Optimistic Concurrency
- Transaction Management
- Command Line Interface

## OR Mapping - Object / Relational Mapping

OR Mapping umfasst immer 3 Komponenten: 

1. Objektgraph / **Entity Type** / Konzeptionelles Modell (im Code, normale .NET Klassen)
2. **Storage Entity** / Datenquelle / Relationales Modell (Tabelle oder Collection)
3. Mapping zwischen 1 und 2 

### Entity Type

Entspricht einer normalen Klasse im .NET Code, beinhaltet

- Properties
- Entity Keys und Alternate Keys
- Relationships/Associations
- Foreign Keys

Eine **Entity** bezeichnet einen strukturierten Datensatz mit einem Key (OO: Objekt). 

Gruppiert in Entity Sets, welche alle Instanzen eines Entity Types (OO: Klasse) beinhalten.

Speziell für OR Mapping: Entitäten können voneinander erben, ungleich in relationalen Datenbanken.

Eine **Relationship** oder Association defiiniert eine Beziehung. Besitzt jeweils zwei Enden mit Kardinalitäten, Abgebildet durch Navigation Properties oder Association Sets (EF spezifisch) bzw. Foreign Keys.

### Storage Entity

Kann eines von folgenden sein

- Tabelle in einer relationalen DB
- View
- Stored Procedures
- (Graph, Collection, ...)

, und beinhaltet *Columns, Primary Keys, Unique Key Constraints, Foreign Keys*.

### Mapping

Mapping von Entities, Property zu Column, etc.
Muss nicht immer 1:1 sein.

Erweiterte Szenarien:

- Vererbung
- Table Split, Merge möglich
- Complex Types (Bsp. Kapselung von Adresse als eigenes Objekt innerhalb der Customer Tabelle)

## 3 verschiedene  Ansätze zum Mapping von Objekten

### Database First

Generiert das Mapping und die Objekte auf Basis einer bestehenden Datenbank. Wird schon seit Anfang von EF und EF Core unterstützt.

Benötigt zusätzliche Projektreferenzen mit `dotnet add package `

​		`Microsoft.EntityFrameworkCore.Tools `

​		`Microsoft.EntityFrameworkCore.Design `

​		`Microsoft.EntityFrameworkCore.SqlServer`

Scaffolding des Codes ausführen mit folgendem Command (`-o` erstellt die Code Klassen in einen neuen Unterordner)

`dotnet ef dbcontext scaffold <connectionstring> <provider als NuGet Paket> -o Models `

### Model First

Generiert die Datenbank, das Mapping und die Objekte basierend auf einem Model (in EF Core nicht verfügbar, hier nicht relevant)

### Code First

Generiert das Mapping und die Datenbank auf Basis des Codes (EF 5.0+, EF Core 1.0+), unter Verwendung der 3 verschiedenen Syntaxen.

| Ansatz        | Funktionsweise                                               |
| ------------- | ------------------------------------------------------------ |
| By Convention | Verwendet Klassenname pluralized: <br />Klasse `Category` -> Tabelle `Categories`<br />Property wird nach Name übernommen |
| By Attributes | Attribut auf Klasse / Property definiert:<br />`[Table("Categories")]`<br />`[Key, Column("Id")]` |
| By Fluent API | Verwendet `ModelBuilder` Objekt in überschriebener Methode `OnModelCreating`<br />`[...].Entity<Category>().ToTable("Categories")`<br />`.HasKey(c => c.Id).HasColumnName("Id")` |

## Mapping  - Providerunabhängig

### Include/Exclude von Entities 

```csharp
modelBuilder.Entity<Category>();		
modelBuilder.Ignore<Metadata>();		// Fluent API
[NotMapped]							  // Data Annotations
public class Metadata { /* ... */ }
```

### Include/Exclude von Properties

Convention: Alle public Properties mit getter/setter werden gemappt.

```csharp
modelBuilder.Entity<Category>()		
    .Property(e => e.Name);
modelBuilder.Entity<Category>()
    .Ignore(e => e.LoadedFromDatabase); // Fluent API
public class Category
{
    [NotMapped] 				// Data Annotations
    public string LoadedFromDatabase { get; set; }
}
```

### Keys

Convention: Property mit dem Namen `[Entity]Id`

```csharp
modelBuilder.Entity<Category>()		// Fluent API
    .HasKey(e => e.Id);				// ODER
	.HasKey(e => new { e.Id, e.Name }) // FluentAPI: einzige Variante für Composite Keys
public class Category
{
    [Key] 						  // Data Annotations
    public int Id { get; set; }
}
```

### Required / Optional

Convention: Value Types werden NOT NULL, Nullable Value Types und Reference Types werden NULL.

```csharp
modelBuilder.Entity<Category>()		// Fluent API
    .Property(e => e.Name)
    .IsRequired()					// ODER
    .IsOptional();
public class Category
{
    [Required]					  // Data Annotations
	public string name { get; set; }
}
			
```

### String Length

Convention: Keine Restriktion auf normalen String Feldern  (`NVARCHAR(MAX)`), 450 Zeichen bei Keys (Primärschlüssel).

```csharp
modelBuilder.Entity<Category>()		// Fluent API
    .Property(e => e.Name)
    .HasMaxLength(200);
public class Category
{
    [MaxLength(200)] 				// Data Annotations
    public string Name { get; set; }
}
```

### Unicode

In Datenbank ist NVARCHAR immer Unicode, ein "normaler" VARCHAR kann nicht by Convention erstellt werden.

```csharp
modelBuilder.Entity<Category>()		// Fluent API
    .Property(e => e.Name)
    .IsUnicode(false);
public class Category
{
    [Unicode(false)] 				// Data Annotations
    public string Name { get; set; }
}
```

### Precision

Verwendet zwei Parameter. Precision: Stellen insgesamt, Scale: Anzahl Nachkommastellen (innerhalb Precision, Angabe optional)

Convention: pro Datentyp vom Provider festgelegt.

```csharp
modelBuilder.Entity<Category>()		// Fluent API
    .Property(e => e.Price)
    .HasPrecision(10, 2);
public class Category
{
    [Precision(10, 2)] 				// Data Annotations
    public decimal Price { get; set; }
}
```

### Indexe

Convention: Werden bei Foreign Keys automatisch erstellt.

```csharp
modelBuilder.Entity<Category>()
    .HasIndex(e => e.Name);			// Fluent API: Non-unique Index
modelBuilder.Entity<Category>()
    .HasIndex(e => new { e.name, e.IsActive }) // Fluent API: Multi-Column Index
    .IsUnique();							// Fluent API: Unique Index

[Index(nameof(Name), nameof(IsActive), IsUnique = true)] 	// Data Annotations (Kombination)
public class Category
{
	public int Id { get; set; }
    public string Name { get; set; }
    public bool? IsActive { get; set; }
}
```

### Entity Type Configuration

Zur Strukturierung von FluentAPI Definitionen kann pro Entity Type jeweils eine Instanz von `IEntityTypeConfiguration<T>` verwendet werden. Somit kann die Definition zur jeweiligen Entity separat geführt werden, und im DbContext wird dann nur noch die entsprechende Instanz aufgerufen.

```csharp
internal class CategoryTypeConfig : IEntityTypeConfiguration<Category> // Separate Definition
{
    public void Configure(EntityTypeBuilder<Category> builder)
    {
        builder
            .Property(p => p.TimeStamp)
            .IsRowVersion();
    }
}
public class ShopContext : DbContext // Zentraler Aufruf im DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new CategoryTypeConfig());
    }
}
```

## Mapping - Relational (MS SQL Server)

### Tabellen

Name der Tabelle zwingend, name des Schemas optional.

```csharp
public class ShopContext : DbContext
{
    // Convention
    public DbSet<Category> Categories { get; set; } 
    
    // Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder) 
    {
        modelBuilder.Entity<Category>().ToTable("Category", schema: "dbo"); 
    }
}
// Data Annotations
[Table("Category", Schema = "dbo")] 
public class Category { public int id; ... }
```

### Spalten

Mapping by Convention: Spaltenname = Property-Name

```csharp
public class ShopContext : DbContext
{
    // Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Category>()
            .Property(e => e.Name)
            .HasColumnName("CategoryName", order: 1);
    }
}
// Data Annotations
public class Category
{
    public int id { get; set; }
    [Column("CategoryName". Oder = 1)]
    public string Name { get; set; }
}
```

### Datentypen / Default Values

Mapping by Convention: Bekannte Datatype Mappings von C# zu Microsoft SQL Server werden implizit ausgeführt.

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\datatype-mappings.png" style="zoom: 33%;" />

```csharp
public class ShopContext : DbContext
{
    // Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Category>()
            .Property(e => e.Name)
            .HasColumnName("CategoryName")
            .HasColumnType("NVARCHAR(500)")
            .HasDefaultValue("undefined");
    }
}
// Data Annotations (keine Default Values möglich)
public class Category{
    public int id { get; set; }
    [Column("CategoryName", TypeName = "NVARCHAR(500)")]
    public string Name { get; set; }
}
```

### Relationships (are complex.)

Auch Assoziation genannt. Wird in der Datenbank mit einer Fremdschlüsselbeziehung dargestellt.

**Principal Entity** beinhaltet den Primärschlüssel, Eltern-Element der Beziehung.
**Dependent Entity** beinhaltet den Fremdschlüssel, Kind-Element der Beziehung.

**Navigation Property** besteht nur im Code, ist eine Referenz auf das andere Objekt zusätzlich zum Key. Kann eine Referenz (Kardinalität 1) oder eine Collection (Kardinalität n) sein. 

**Conventions** Eine Relationship wird erstellt wenn ein Navigation Property auf einem Typ vorhanden ist. Das bedeutet immer dann, wenn ein Property nicht einen "Scalar Type" auf der Datenbank hat, also ein Grundtyp der direkt in ein Feld umgewandelt werden kann.

```csharp
public class Category // Beispiel: Fully Defined Relationship in 3 Teilen
{
    public int Id { get; set; }
    public ICollection<Products> Products { get; set; } // 1/3: Navigation Property
}														// mit n-Kardinalität
public class Product
{
    public int Id { get; set; }
    public int CategoryId { get; set; } // 2/3: Foreign Key
    public Category Category { get; set; } // 3/3: Navigation Property mit 1-Kardinalität
}
```

**Fluent API** 

```csharp
// Fully Defined
modelBuilder.Entity<Post>()
	.HasOne(p => p.Blog)			// Navigation Property auf dieser Entity
	.WithMany(b => b.Posts)			// Inverse Navigation auf gegenüberliegender Entity
    .HasForeignKey(p => p.BlogId)	 // Foreign Key explizit gesetzt
    .HasConstraintName("FK_Post_BlogId");	
```

**Data Annotations**

```csharp
public class Category
{
    public int Id { get; set; }
    public ICollection<Product> Products { get; set; } // Navigation Property 
}													// mit n-Kardinalität
public class Product
{
    public int Id { get; set; }
    public int CategoryId { get; set; }				
    [ForeignKey(nameof(CategoryId))]				// Referenz auf Foreign Key
    public Category Category { get; set; }			// Navigation Property mit 1-Kardinalität
}
```

**Mehrere Relationships** auf dieselbe Zieltabelle

```csharp
public class Post
{ ... }
public class User
{
    public string UserID { get; set; }
    ...
	[InverseProperty("Author")]
	public List<Post> AuthoredPosts { get; set; }
    [InverseProperty("Contributor")]
    public List<Post> ContributedPosts { get; set; }
}
```

**Many-to-Many Relationships** beinhalten auf beiden Seiten ein Collection Navigation Property, wenn sie bei Convention erkannt werden sollen. *Nur mit FluentAPI möglich*.

Wenn im FluentAPI `.UsingEntity(...)` angegeben wird, wird in der Datenbank eine zusätzliche Tabelle mit einem sogenannten **Join Entity Type** erstellt, mit Foreign Keys zu beiden Entities des Relationships. Diese Entity dient zum Halten von Informationen zur Verbindung. Konfiguration zur Join Entity kann via FluentAPI gemacht werden

```csharp
modelBuilder
    .Entity<Post>()
    .HasMany(p => p.Tags)
    .WithMany(p => p.Posts)
    .UsingEntity(j => j.ToTable("PostTags"));	// Join Entity Types
```

**Shadow Foreign Key**: Wenn der Foreign Key nicht explizit definiert ist, wird er automatisch erstellt. 

**Single Navigation Property**: Ein Navigation Property reicht aus, um eine Relationship by Convention zu definieren, *solange es eindeutig ist*. 

```csharp
modelBuilder.Entity<Product>()
    .HasOne<Category>()
    .WithMany(b => b.Products);
```

**On Delete Cascade** ist standardmässig `Cascade Delete` bei required Relationships und `ClientSetNull` für optionale Relationships. ClientSetNull versucht nur, aktuell im Memory geladene dependent entities zu aktualisieren, alle anderen werden nicht verändert.

In FluentAPI kann das DeleteBehavior angepasst werden:

```csharp
modelBuilder.Entity<Product>()
    .HasOne(...).WithMany(...)
    .OnDelete(DeleteBehavior.[Cascasde|SetNull|Restrict])
```

### Computed Columns

Möglich via FluentAPI:

```csharp
modelBuilder.Entity<Category>()
    .Property(e => e.DisplayName)
    .HasComputedColumnSql(
		"[Id] + ' ' + [Name]");
```

