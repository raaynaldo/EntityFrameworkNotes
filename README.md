# Entity Framework Notes

[Entity Framework](https://www.entityframeworktutorial.net/what-is-entityframework.aspx) is an open-source [ORM framework](https://en.wikipedia.org/wiki/Object-relational_mapping "Object-relational Mapping") for .NET applications supported by Microsoft. It enables developers to work with data using objects of domain specific classes without focusing on the underlying database tables and columns where this data is stored. With the Entity Framework, developers can work at a higher level of abstraction when they deal with data, and can create and maintain data-oriented applications with less code compared with traditional applications.

> Always work on small changes and small migrations for reduce the change of error.
>
> Create another Migration, don't delete the migration versioning is always forward.

## Command

Initials code for EntityFramework

    install-package EntityFramework

    enable-migration

    add-migration <nameofmigration>
    add-migration <nameofmigration> -IgnoreChanges -Force
    add-migration <nameofmigration> -Force

    update-database
    update-database -TargetMigration: <nameofmigration>

## Migration.cs

    public override void Up()
    {
        //Run When Migration update
    }

    public override void Down()
    {
        //Run when Migration downgrade
    }

You should pay attention with Up also Down too.

    Sql("<querywhere>");
    Make identity:false => Id = c.Int(nullable: false, identity: false),
    Sql("INSERT INTO Categories VALUES (1, 'Web Development')");
    Sql("INSERT INTO Categories VALUES (2, 'Programming Languanges')");

    Make identity:true => Id = c.Int(nullable: false, identity: false),
    Sql("INSERT INTO Categories (Name) Web Development");
    Sql("INSERT INTO Categories (Name) Programming Languanges");

### MakeNullableFalseInColumn

    AddColumn("dbo.Courses", "Name", c => c.String(nullable:false));

### RenameColumnName

This Code will delete all data in column Title :

>     AddColumn("dbo.Courses", "Name", c => c.String());
>     DropColumn("dbo.Courses", "Title");

You better using This :

> `RenameColumn("dbo.Courses", "Title", "Name");`

or

>     AddColumn("dbo.Courses", "Name", c => c.String());
>     Sql("UPDATE Courses SET Name = Title");
>     DropColumn("dbo.Courses", "Title");

## Seeding

    protected override void Seed(CodeFirstExistingDatabase.PlutoContext context)
    {
        context.Authors.AddOrUpdate(a => a.Name,
            new Author
            {
                Name = "Author 1",
                Courses = new Collection<Course>()
                {
                    new Course() { Name = "Course for Author 1", Description = "Description" }
                }
            });
    }

## Deployment

You’re ready to deploy the application. Your DBA expects you to provide a database script. Run the following command to get a SQL script of all your migrations:

> `Update‐Database ­‐Script ‐SourceMigration:0`

This command generates the SQL script from the very ﬁrst migration to the last one. In a real-world scenario, you may want to change the range of migrations included in the SQL script in each deployment. To do this, you can use:

> `Update-­Database ‐Script ‐SourceMigration:Migr1 ‐TargetMigration:Migr2`

## Data Annotations vs Fluent API

| Data Annotations        | Fluent API       |
| ----------------------- | ---------------- |
| Code in attribute class | Code in function |

> It's better choose one of that approach, don't use both. It will complicate maintenance the application.
>
> It's better using Fluent API its more powerful than Data Annotations approach.
> We can configure all of relationship using fluent API

**Data Annotations**
[Required] --> Data Annotations
public string Name { get; set; }

**Fluent API**
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
//
}

List of Fluent API Code :

- Basic

  ```c#
  modelBuilder.Entity<Course>()
  ```

- Change Table Names

  ```c#
  modelBuilder.Entity<Course>().ToTable("<newtablename>");
  modelBuilder.Entity<Course>().ToTable("<newtablename>","<schemaname>");
  ```

- Primary Keys

  ```c#
  modelBuilder.Entity<Book>().HasKey(t => t.ISBN);

  //Composite Key(Combination of more than one PK)
  modelBuilder.Entity<Course>().HasKey(t => new {t.OrderId, t.OrderItemId});
  ```

- Column Names

  ```c#
  modelBuilder.Entity<Course>()
    .Property(t => t.Name)
    .HasColumnName("<newcolumnname>")
  ```

- Column Types

  ```c#
  modelBuilder.Entity<Course>()
    .Property(t => t.Name)
    .HasColumnType("<newcolumnname>")
  ```

- Column Orders

  ```c#
  modelBuilder.Entity<Course>()
    .Property(t => t.Name.HasColumnOrder(<numberofcolumnorder>)
  ```

- Database Generated

  ```c#
  modelBuilder.Entity<Book>()
    .Property(t => t.ISBN.HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);

  //DatabaseGeneratedOption.None
  //DatabaseGeneratedOption.Identity
  //DatabaseGeneratedOption.Computed
  ```

- Nulls

  ```c#
  modelBuilder.Entity<Course>()
    .Property(t=>t.Name)
    .IsRequired();
  ```

- Length of Strings

  ```c#
  modelBuilder.Entity<Course>()
    .Property(t=>t.Name)
    .HasMaxLength("<lengthofstring>");

  modelBuilder.Entity<Course>()
    .Property(t=>t.Description)
    .IsMaxLength();
  ```

- Relationships

        Type1 -> Type2
            HasMany()
            HasRequired()
            HasOptional()
        Type1 <- Type2
            WithMany()
            WithRequired()
            WithOptional()

  - One to Many
    ```c#
    // Author 1 -> * Course
    modelBuilder.Entity<Author>()
        .HasMany(a=>a.courses)
        .WithRequired(c=>c.Author)
        .HasForeignKey(c=>c.AuthorId)
    ```
  - Many to Many
    ```c#
    // Course * -> * Tag
    modelBuilder.Entity<Course>()
        .HasMany(c=>c.Tags)
        .WithMany(t=>t.Courses)
        //We don't need make that code bcoz entity automaticly create many-to-many
        .map(m=>m.ToTable("CourseTags"))
        //but for changes the table name we need make that.
    ```
  - One to Zero/One
    ```c#
    // Course 1 -> 0..1 Cover
    modelBuilder.Entity<Course>()
        .HasOptional(c=>c.Caption)
        .WithRequired(c=>c.Course)
    ```
  - One to One
    ```c#
    // Course(Principal/Parent) 1 -> 1 Cover(Dependent/Child)
    modelBuilder.Entity<Cover>()
        .HasRequired(c=>c.Course)
        .WithRequiredDependent(c=>c.Cover)
    ```

## Organizing Fluent API Configurations

make a new class

```c#
public class CourseConfiguration : EntityTypeConfiguration<Course>
{
    public CourseConfiguration()
    {
        //Copy all Course Configuration in OnModelCreating and remove the modelBuilder.Entity<Course>()
    }
}
```

```c#
//in OnModelCreating

protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Configuration.Add(new CourseConfiguration());
}
```

## Eager Loading

```c#
// For single properties
context.Courses.Include(c=>c.Author.Address);

// For collection properties
context.Courses.Include(a=>a.Tags.Select(t=>t.Moderator));
```

## Explicit Loading

```c#
var author = context.Authors.Single(a => a.Id == 1);

context.Courses.Where(c => c.AuthorId == author.Id &&
c.FullPrice == 0)// We can apply filter
.Load(); // Load for Explicit Loading

var authorIds = context.Authors.Select(a=> a.Id);

context.Courses.Where(c => authorsIds.Contains(c.AuthorId) && c.FullPrice == 0).Load();
```

## Adding Objects

- Using an existing object in context

  ```c#
  var context = new PlutoContext();

  var author = context.Authors.Single(a => a.Id == 1);
  var course =  new Course
  {
      Name = "New Course",
      Description = "New Description",
      FullPrice = 19.95f,
      Level = 1,
      Author = author // using existing object
  }
  context.Courses.Add(course);

  context.SaveChanges()
  ```

- Using foreign key properties

  ```c#
  var context = new PlutoContext();

  var course =  new Course
  {
      Name = "New Course",
      Description = "New Description",
      FullPrice = 19.95f,
      Level = 1,
      AuthorId = 1 // using foreign key
  }
  context.Courses.Add(course);

  context.SaveChanges()
  ```

## Updating Objects

```c#
var context = new PlutoContext();

var course = context.Courses.Find(4);
couse.Name = "New Name";
couse.AuthorId = 2;

context.SaveChanges();

```

### Removing Objects

- With Cascade Delete

  ```c#
  var context = new PlutoContext();

  var course = context.Courses.Find(6);
  context.Courses.Remove(course);

  context.SaveChanges();

  ```

- Without Cascade Delete

  ```c#
  var context = new PlutoContext();

  var author = context.Authors.Inculde(a => a.Courses).Find(2);
  context.Courses.RemoveRange(author.Courses);
  context.Courses.Remove(author);

  context.SaveChanges();

  ```
