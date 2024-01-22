# Entity Framework Core

### Setup

Simple class:
```C#
public class Author
{
    public int AuthorId { get; set; }
    public string? FirstName { get; set; }
    public string? LastName  { get; set; }
    public DateOnly Contracted { get; set; }
    public int BookCount { get; set; }
    public bool Pseudonym { get; set; }
    public GenresEnum CoreGenre { get; set; }
    public decimal RoyaltyRate { get; set; }
}
```

The above class maps to any database implementation automatically, as long as the database provider supports it. Here is what the `Author` class turns into as a SQL Server table:
```SQL
AuthorId     int       not null
FirstName    nvarchar  (max) null
LastName     nvarchar  (max) null
Contracted   date      not null
BookCount    int       not null
Pseudonym    bit       not null
CoreGenre    int       not null
RoyaltyRate  decimal   (18,2) not null
```

EF Core will also automatically assign a field named `Id` or `[ClassName]Id` is automatically set as the primary key.

The DbContext drives everything:
```C#
using Microsoft.EntityFrameworkCore;

public class BigPictureContext : DbContext
{
    public DbSet<Author> Authors => Set<Author>();
}
```

Note that the `Author` class would have named the table "Author" by default, but the name given in the DbContext overrides that. In this case the table will be named "Authors", which fits the table convention of having table names be plural.

EF Core can also comprehend table relationships. Here is one way to describe a 1-to-Many relationship between an Author and their books:
```C#
public class Author
{
    public int AuthorId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; }

    public Author(string first, string last)
    {
        FirstName = first;
        LastName = last;
        Books = new List<Book>();
    }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public Author? Author { get; set; }

    public Book(string title)
    {
        Title = title;
    }
}
```

EF automatically creates a foreign key in the Book table called `AuthorId`. By default, EF Core does things behind the scenes that "most people would want". In this case, a "Cascade Delete Rule" was created, so if an Author is ever deleted from the database, *all of that authors books are automatically deleted as well*. Naturally, these defaults can be overridden if needed.

The updated DbContext:
```C#
public class BigPictureContext : DbContext
{
    public DbSet<Author> Authors => Set<Author>();
    public DbSet<Book> Books => Set<Book>();
}
```

Consider a case where a Book can have multiple authors. Now we have a Many-to-Many relationship. This can be represented as follows:
```C#
public class Author
{
    public int AuthorId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public List<Book> Books { get; set; }

    public Author(string first, string last)
    {
        FirstName = first;
        LastName = last;
        Books = new List<Book>();
    }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public List<Author> Authors { get; set; }

    public Book(string title)
    {
        Title = title;
        Authors = new List<Author>();
    }
}
```

EF Core infers the Many-to-Many relationship here, and automatically creates a join table under the hood.

### Execution

EF Core has an extension of LINQ that allows for querying the database. Here is a simple query:
```C#
var context = BigPictureContext();
List<Author> authors = context.Authors.ToList();
```

The above generates the following SQL:
```SQL
SELECT [a].[AuthorId],
    [a].[FirstName],
    [a].[LastName]
FROM [Authors] AS [a]
```

Filter, Aggregate, and Parameterize:
```C#
var firstLetter = "L";
context.Authors
    .Where(a => a.LastName.StartsWith(firstLetter))
    .FirstOrDefault();
```

`.Where()` is self explanatory. `.FirstOrDefault()` triggers the query, and returns the first result found in the database.

The above generates the following SQL:
```SQL
(Incoming parameter: @__firstLetter_0 nvarchar(4000)', @__firstLetter_0=N'L)
SELECT TOP(1) [a].[AuthorId]
    [a].[FirstName]
    [a].[LastName]
FROM [Authors] AS [a]
WHERE (@__firstLetter_0 = N'''') --handles a null parameter
    OR (LEFT([a].[LastName], LEN(@__firstLetter_0)) = @__firstLetter_0)
```

Note that some apostrophes are off because they are in a different unicode than the screenshot I was referencing, but the gist of what is going on is there. EF also accounts for SQL injection.

To join tables, use `.Include()` for eager loading:
```C#
context.Authors.Include(a => a.Books).ToList();
```

Query methods:
```C#
Where()
OrderBy()
Skip()
Take() // Skip + Take for paging
Join()
GroupJoin()
GroupBy()
Select() // for projection
SelectMany()
```

Execution methods (all with Async counterparts, which you should be using):
```C#
ToList()
First()
FirstOrDefault()
Single()
SingleOrDefault()
Last()
LastOrDefault()
Count()
LongCount()
Min()
Max()
Average()
Sum()
AsEnumerable()

// new as of EF 7
ExecuteUpdate()
ExecuteDelete()
```

A note on `Include()` vs `Join()`:

`Include()` is likely the behavior you're looking for when combining fields from different tables; `Include()` returns an IEnumerable of the requested type, whose structure matches the C# model:
```
Author1
    Book1
    Book2
    Book3
Author2
    Book4
    Book5
```

`Join()` on the other hand projects into an IEnumerable of an anonymous type with a flat structure, in this case of type `IEnumerable<Anonymous<Author, Book>>`:
```
{ Author1, Book1 }
{ Author1, Book2 }
{ Author1, Book3 }
{ Author2, Book4 }
{ Author2, Book5 }
```

### Change Tracking

EF Core uses "change tracking" to maintain details of the state of each object that it is aware of. EF Core then uses this tracking information construct `INSERT`, `UPDATE`, and `DELETE` statements to send to the database.

Tracker details for each object:
- Object State:
  - Added
  - Modified
  - Deleted
  - Unchanged
- Original property values
- Current property values

`DbContext.SaveChanges()` constructs SQL based on the above tracker details. When `DbContext.SaveChanges()` is called:
- It forces context to update state info for each tracked object
- It constructs the relevant SQL based on the state of each entity
- It opens a db connection (or depending on your connection string, finds an existing one) and executes each command (using bulk operations and database transactions)
- It returns the number of affected and new primary keys
- It updates entity primary keys and foreign keys for tracked objects that are still in memory
- The change tracker reinitializes

EF is smart enough to optimize certain database calls. For example, if only a single property is updated on a tracked author object, the SQL query EF generates will only update a single record.

## Applying Mappings

In some cases, the database and C# classes won't look exactly alike. Mappings tell EF how to properly connect everything together.

Say for example that there's a field in the database called "lname_or_surname". You'll likely not want to call your class property that, so we can apply a mapping in two different ways:

Fluent API (defined in the DbContext class):
```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
        .Property(a => a.LastName)
        .HasColumnName("lname_or_surname");
}
```

Data Annotations (defined in the class)
```C#
[Column("lname_or_surname")]
public string LastName { get; set; }
```

Because all mappings are available in the Fluent API, it is the recommended method. It also keeps EF-specific information outside of the business logic.

Some more simple mapping configurations:
```C#
// overrides EF's assumptions about the PK name
modelBuilder.Entity<Author>()
    .HasKey(a => a.AuthorPK)

// alert EF about properties that map to computed columns so that EF does not try and push data into them
modelBuilder.Entity<Book>()
    .Property(b => b.PublishDate)
    .HasComputedColumnSql();

// EF will not search or query for certain classes or properties
modelBuilder.Entity<Book>()
    .Ignore(b => b.CommissionRate)
```

Instead of odd scenarios with the database, EF manages to handle odd scenarios with the classes as well. Say you would like to use the field of a given property to populate a table's field instead of the property itself:
```C#
public class Author
{
    // ...
    private string? _taxId;
    public void SetTaxId(string taxId)
    {
        _taxId = taxId;
    }
}

// in DbContext
modelBuilder.Entity<Author>().Property("_taxId").HasColumnName("TaxId");
```

EF also let's you store data that you don't want in your classes using "shadow properties". Examples of very popular shadow properties are fields to store created and modified timestamps. We often like to have these in our database, but don't need those fields stored in our models. To do this, add this to the DbContext:
```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>().Property<DateTime>("Aud_Created"); // "Aud" means "Audit"
    modelBuilder.Entity<Author>().Property<DateTime>("Aud_Modified");
}

public override int SaveChanges()
{
    var timestamp = DateTime.Now;
    // filter on new or updated authors
    var EntriesToModify = ChangeTracker.Entries()
                                       .Where(e => e.Entity is Author
                                              && (e.State == EntityState.Added|| e.State == EntityState.Modified))

    foreach (var entry in EntriesToModify)
    {
        entry.Property("Aud_Modified").CurrentValue = timestamp;
        if (entry.State == EntityState.Added)
        {
            entry.Property("Aud_Created").CurrentValue = timestamp;
        }
    }

    return base.SaveChanges();
}
```

EF's other complex mapping capabilities:
- DDD (domain driven design) value objects capable via complex type support
- Define mapping relationships solely in DbContext for when there are more complex scenarios than the 1-to-Many and Many-to-Many defaults
- Bulk configurations. For example, if you need to apply the same mapping across all entities in your data model (e.g. overriding all strings to map to `varchar(100)`)

### Using Your Own SQL

EF can call views (which can be generated based off of your model classes) and stored procedures.

Stored procedure example in `Program.cs`:
```C#
RawSqlStoredProc();
void RawSqlStoredProc()
{
    var authors = _context.Authors
        .FromSqlRaw("AuthorsPublishedInYearRange {0}, {1}", 2010, 2015) // no string interpolation
        .ToList();
}

InterpolatedSqlStoredProc();
void InterpolatedSqlStoredProc()
{
    int start = 2010;
    int end = 2015;
    var authors = _context.Authors
        .FromSql($"AuthorsPublishedInYearRange {start} {end}") // string interpolation
        .ToList();

        // note: EF 6 and below uses the method FromSqlInterpolated
}
```

EF 7 has made some important updates; initially, if you wanted to delete or update anything you would first have to retrieve it from the database, or call a stored procedure. EF 7 has added the `ExecuteDelete()` and `ExecuteUpdate()` methods (and their async counterparts), which prevent from having to do this:
```C#
var rowCount = _context.Books.Where(b => b.BookId == 1).ExecuteDelete();

var rowCount = await _context.Books
    .Where(p => p.publishedOn.Year < 2000)
    .ExecuteUpdateAsync(s => s.SetProperty(b => b.Price, 1.50));
```

### Performance

- Remove tracking from entities when not needed
- Synchronous database operations cause blocking
  - Always use the Async methods
  - Use `SaveChangesAsync()`
- See [this link](https://learn.microsoft.com/en-us/ef/core/performance/) for more
  - How modeling affects performance
  - Debugging performance issues
  - Etc

### Logging

Simple logging:
```C#
optionsBuilder.LotTo(Console.WriteLine);
```

ASP.NET Core appsettings.json (and via AddDbContext):
```C#
"Logging": {
    "LogLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
}
```

EF Core captures:
    - SQL
    - ChangeTracker activity
    - Interaction with database
    - Database transactions  

EF Core specific configurations:
    - EnableDetailedErrors
    - EnableSensitiveData (by default, data that is sent as parameters to queries is not logged unless this is enabled)
    - Filter based on message type (e.g. database messages)
    - Even more detailed filtering can be found in the docs

Tap into EF Core pipeline
    - Override `SaveChanges()` (execute logic before or after the save is executed)
    - Event handlers for Change Tracking methods
    - Interceptors to intercept an action (e.g. commands or results), and then either change the action or let it continue.
      - IDbCommandInterceptor
        - Before sending a command to the database
        - After the command has executed
        - Command failures
        - Disposing the command's DbDataReader
      - IDbConnectionInterceptor
        - Opening and closing connections
        - Connection failures
      - IDbTransactionInterceptor
        - Creating transactions
        - Using existing transactions
        - Committing transactions
        - Rolling back transactions
        - Creating and using savepoints
        - Transaction failures

### Handling Database Connections and Transactions

- EF will auto retry a connection or command to a database depending on the error code returned by the database.
- If `SaveChanges()` is sending multiple commands, they will always be wrapped in a transaction, and rolled back on any failure.
- Distributed transactions are supported (.NET 7+)