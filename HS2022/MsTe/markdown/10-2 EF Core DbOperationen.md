# Datenbankoperationen

## Database-Context

Das Objekt `DbContext` ist vorgesehen, um jeweils eine *Unit of Work* auszuführen. Die Lifetime des Objekts ist damit jeweils sehr kurz. Änderungen auf Entities werden nachverfolgt und erst mit dem Aufruf von `context.SaveChanges()` oder `context.SaveChangesAsync()` in die Datenbank geschrieben.

Konzeptionell werden die beiden *Design Patterns Repository* und *Unit of Work* umgesetzt.

Funktionalität **zur Design Time**: Model Definieren, Konfigurationen, Database Migrations,

Funktionalität **zur Runtime**: Connections verwalten (Pooling), CRUD Operationen ausführen, Change Tracking, Caching, Transaction Management

Do's:

- innerhalb einem *using*-Statement verwenden
- bei Web Applikationen eine Instanz pro Request
- im GUI eine Instanz pro Formular

Don'ts: 

- Instanz zu lange leben lassen (Limitierte Anzahl Connections im Client Connection Pool, Change Tracking hat Grenzen in der Effizienz)
- Instanz sharen (ist nicht Thread-Safe, Exception an einem Ort kann gesamte Instanz unbrauchbar machen)

```csharp
using (ShopContext context = new()) // Context Instanzieren, Change Tracker initialisieren
{ 
    Category category = await context     // Abfragen mit 'LINQ to Entities'
        .Categories
        .SingleAsync(c => c.Id == 1);
    category.Name = $"{category.Name} / Changed";	// Daten anpassen
    await context.SaveChangesAsync();			   // Sync zurück in DB
    
    var categories = context.Categories;
    foreach (Category c in categories) { Console.WriteLine(c.Name); } // Deferred Abfrage
} // Context schliessen: Cache invalidieren, DB-Verbindung zurück in Connection Pool
```

### LINQ To Entities

Entity Framework generiert die Queries, Datenbank führt diese aus. LINQ to Entities Queries werden zu Expression Trees kompiliert. Nicht alle .NET Expressions können auf Datenbank-Syntax übersetzt werden. Der generierte SQL Output ist je nach Formulierung unterschiedlich, nicht immer optimal.

### Insert

```csharp
using ShopContext context = new();
Category cat = new() { Name = "Notebooks" };
// 3 Alternative Varianten um die neue Kategorie hinzuzufügen
context.Add(cat);
context.Categories.Add(cat);
context.Entry(cat).State = EntityState.Added;
await context.SaveChangesAsync();				// Führt SQL aus
```

### Update

```csharp
using ShopContext context = new();
Category cat = await context
    .Categories
    .FirstAsync();
cat.Name = "Changed";						   // Change im Memory
await context.SaveChangesAsync();				// Führt SQL aus
```

### Delete

```csharp
using ShopContext context = new();
Category cat = await context
    .Categories
    .FirstAsync(c => c.Name == "Notebooks");
// 3 Alternative Varianten zum Löschen
context.Remove(cat);
context.Categories.Remove(cat);
context.Entry(cat).State = EntityState.Deleted;
await context.SaveChangesAsync();				// Führt SQL aus
```

### Multiple Operations

```csharp
using ShopContext context = new();
Category cat1 = new { Name = "Notebooks" };
context.Add(cat1);

Category cat2 = new { Name = "Displays" };
context.Add(cat2);

Category cat3 = await context
    .Categories
    .SingleAsync(c => c.Id == 1);
cat3.Name = "Changed";
await context.SaveChangesAsync();			// Führt SQL aus
```

### CUD von Assoziationen

```csharp
order.Customer = customer; 		// Anpassen von Navigation Properties
customer1.Orders.Add(order);	// Hinzufügen / Entfernen von Elementen in Collection Navigation Properties
customer2.Orders.Remove(order);
order.CustomerId = 1;		// Setzen des Foreign Keys (einzige Variante, die keine weiteren DB-Zugriffe macht)

```

## Context - Change Tracking

Change Tracker registriert alle Änderungen an Entities die aus der DB geladen werden. Agiert komplett offline, keine Live-Checks auf der Datenbank. Die möglichen States sind *Added, Deleted, Modified, Unchanged*.

`context.Add(cat);`
`context.Categories.Add(cat);  `berücksichtigen ganzen Objekt-Graphen,
`context.Entry(cat).State = EntityState.Added;  `  nur die jeweilige Entity.

State Entries: beinhaltet Informationen über

- Status des Objekt
- Info pro Property: Geändert (boolean), Originalwert, Aktueller Wert
- Entity-spezifische Ladefunktionen
- "Load Referenced Entities"

## Optimistic Concurrency

Basiert auf der Annahme, dass ein Objekt zwischen Laden und Speichern eines Records nicht anderweitig verändert wurde. Dies wird vor dem Speichern eines Datensatzes geprüft.

*Alternative Pessimistic Concurrency:* Datensatz wird für die Dauer der Verarbeitung gesperrt. Probleme dabei sind Deadlocks, Orphaned Locks und schlechte Performance.

Die Erkennung von Konflikten beim Speichern kann mit zwei Varianten erfolgen, Timestamp oder ConcurrencyToken. Beide Varianten laden beim Lesen des Datensatzes weitere Informationen mit, die beim Schreiben zum Vergleich wieder verwendet werden können.

### Timestamp oder Row Version

SQL vesucht, eine Änderung in die Datenbank zu schreiben, und prüft in der Where Clause auf den Timestamp. `@@ROWCOUNT` gibt an, wie viele Datensätze vom letzten SQL Statement verändert wurden. Wenn der Wert 0 ist, weiss Entity Framework, dass ein Concurrency Fehler passiert ist und wirft eine entsprechende Exception.

```sql
UPDATE [categories] SET [name] = @p0
WHERE [Id] = @p1 AND [Timestamp] = @p2; /* Timestamp oder Versions-Check */
SELECT [Timestamp] FROM [Categories]
WHERE @@ROWCOUNT == 1 AND [Id] = @p1
```

Das Feld, welches im Code als Timestamp oder RowVersion definiert wird, muss ein automatisches Inkrementieren bei jeder Änderung unterstützen. 

```csharp
modelBuilder.[...]
	.Property(p => p.Timestamp)
    .IsRowVersion();			// Fluent API
[Timestamp] 					// Data Annotation
public byte[] Timestamp { get; set; }
```

### ConcurrencyToken

Ähnlich wie beim Timestamp werden gewisse Spalten definiert, deren Wert vor dem Speichern geprüft wird. 

```csharp
modelBuilder.[...]
    .Property(p => p.Name)
    .IsConcurrencyToken(); 		 // Fluent API
[ConcurrencyCheck] 				// Data Annotations
public string Name { get; set; }
```

Sind diese noch gleich wie zum Zeitpunkt des Lesens, wird das Update gemacht. Andernfalls wird von Entity Framework genauso im SQL via `ROWCOUNT` auf Erfolg oder Fehler geprüft.

```sql
UPDATE [Products] SET [NAME] = @p0
WHERE [Id] = @p1
AND [Name] = @p2 AND [Price] IS NULL; /* Concurrency Token Check */
```

### Varianten zur Konfliktbehandlung

Exception beinhaltet sämtliche wichtigen Informationen: Aktuelle Werte, Original-Werte, Datenbank-Werte.

- Exception fangen, beginne von vorne
- Benutzer Fragen, Vergleich von allen Werten
- Versuch einer Autokorrektur
- Blind überschreiben (nogo!)

```csharp
try { /* ... */ }
catch (DbUpdateConcurrencyException ex)
{
    foreach (EntityEntry entry in ex.Entries)
    {
        if (entry.Entity is Product)
        {
            var proposedValues = entry.CurrentValues;
            var databaseValues = await entry.GetDatabaseValuesAsync();
            foreach (IProperty property in proposedValues.Properties)
            {
                object proposedValue = proposedValues[property];
                object databaseValue = databaseValues[property];
                // TODO: decide which value should be written to table
                // proposedValues[property] = <value to be saved>;
            }
            // Refresh original values to bypass next concurrency check
            entry.OriginalValues.SetValues(databaseValues);
        }
        else { /* ...*/ }
    }
}
```

## Ladestrategien

### Eager Loading (Default)

Assoziationen werden nicht direkt geladen, Include()-Statement nötig. Generiert ein JOIN Statement in derselben SQL Abfrage. Queries werden schnell kompliziert.

```csharp
Order order = await context
	.Orders
	.Include(o => o.Customer)	// eager loading
	.Include(o => o.Items)
		.ThenInclude(oi => oi.Product)	// cascaded eager loading
	.FirstAsync();							// << DB Abfrage
var customer = order
	.Customer; // customer is not "null"
var items = order
	.Items; // items is not "null"
```

### Explicit Loading

Assoziationen werden explizit nachgeladen, Collections werden komplett geladen. Pro nachladen separate SQL Abfrage.

```csharp
Order order = await context
    .Orders
    .FirstAsync();				// << DB Abfrage
await context
    .Entry(order)
    .Reference(o => o.Customer)		// Parents
    .LoadAsync();				// << DB Abfrage
await context
    .Entry(order)
    .Collection(o => o.Items)		// Collections
    .Query()
    .Where(oi => oi.QuantityOrdered > 1) // Filter (inkl. .Query() voraus)
    .LoadAsync();
```

### Lazy Loading

Assoziationen werden bei Zugriff auf Property automatisch nachgeladen, Collections komplett aber mit FIltermöglichkeit. Separate Abfrage. **Problem:** Logik fürs Nachladen muss auf dem Property im `get` implementiert sein. Dies entweder manuell mittels LazyLoader (Dependency Injection) oder automatisch mit `Microsoft.EntityFrameworkCore.Proxies`.

```csharp
Order lazyOrder;
using (ShopContext context = new())
{
    lazyOrder = await context
        .Orders
        .FirstAsync();					// << DB Abfrage
    var customer = lazyOrder.Customer;	 // << DB Abfrage
    var items = lazyOrder.Items;		// << DB Abfrage 
    var items2 = lazyOrder.Items;
}
var fail = lazyOrder.SomeProperty;		// Nachladen nur mit aktivem DbContext möglich
```

Implementation mit Proxies:

```csharp
public class ShopContext : DbContext // zwingend public, NICHT sealed
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    { optionsBuilder.UseLazyLoadingProxies(); }
}
public class Order	// zwingend public, NICHT sealed
{
    public int Id { get; set; }
    public virtual Customer Customer { get; set; }
    public virtual ICollection<OrderItem> Items { get; set; }
}
```

## Anhang

### LocalDB Connection

`SqlLocalDB.exe create AngProjDb`, und im File `AppSettings.json` den Connection String hinterlegen:

```json
{
  "ConnectionStrings": {
    "AngProjDatabase": "Server=(localdb)\\AngProjDb;Database=AngProj;Trusted_Connection=True;"
  }
}
```

Im SQL Server Management Studio kann mit folgendem Servernamen auf die Datenbank verbunden werden: `(LocalDb)\AngProjDb`

<img src="C:\Users\luzia\OneDrive - OST\01_Aktuelles_Semester\MsTe\res\ssms-localdb.png" style="zoom:50%;" />