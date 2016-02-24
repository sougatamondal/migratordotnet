# Some Example Migrations #

The following are some common scenarios.
Hopefully we can support more in the future.

**Note**
Prior to Migrator.Net 0.8 the Migration annotations took incrementing integer values.
From Migrator.Net 0.8 and greater it is recommended that you use timestamp values. Timestamps will allow you to more easily do development using branches without creating conflicting Migration numbers.

## A Sample DB Independent Migration ##

Migrations that don't directly execute SQL should be able to be run on any database. The current Provider will will be set in the script or from the Command Line and it will format the proper SQL for that database implementation.

```
using Migrator.Framework;
using System.Data;

[Migration(20080805151231)]
public class AddCustomerTable : Migration
{
        public override void Up()
        {
                Database.AddTable("Customer",
                          new Column("name", DbType.String, 50),
                          new Column("address", DbType.String, 100),
                          new Column("age", DbType.Int32, 100)
                         );
        }
        public override void Down()
        {
                Database.RemoveTable("Customer");
        }
}
```

## Compound Primary Keys ##

The manual way is to create a table with no primary keys and then declare the primary keys separately.

```
using Migrator.Framework;
using System.Data;

[Migration(20080806151301)]
public class AddCustomerTable : Migration
{
        public override void Up()
        {
                Database.AddTable("CustomerAddress",
                          new Column("customer_id", DbType.Int32),
                          new Column("address_id", DbType.Int32)
                         );
                Database.AddPrimaryKey("CustomerAddress", "customer_id", "address_id");
        }
        public override void Down()
        {
                Database.RemoveTable("CustomerAddress");
        }
}
```

The easier way is just to declare each of the columns as a primary key and let the tool do the work for you.

```
using Migrator.Framework;
using System.Data;

[Migration(20080806161420)]
public class AddCustomerTable : Migration
{
        public override void Up()
        {
                Database.AddTable("CustomerAddress",
                          new Column("customer_id", DbType.Int32, ColumnProperty.PrimaryKey),
                          new Column("address_id", DbType.Int32, ColumnProperty.PrimaryKey)
                         );
        }
        public override void Down()
        {
                Database.RemoveTable("CustomerAddress");
        }
}
```


## A Sample Migration With different SQL for Different Databases ##

Only the parts that match Database[**current provider**] will be executed. This allows you to do things that are not database independent but still support multiple databases if necessary.

In the below example we are supporting SqlServer and PostgreSQL and only the proper script will be run depending on the current provider executing. e.g. If your NAnt script sets the provider to SqlServer then the SqlServer blocks will be executed.

```
[Migration(20080806160101)]
public class AddFooProcedure : Migration
{
        public override void Up()
        {
                Database["SqlServer"].ExecuteNonQuery(
@"CREATE PROCEDURE Foo 
	@var int = 0
AS
BEGIN
	SELECT @var
END");

                Database["PostgreSQL"].ExecteNonQuery(
@"create or replace function foo() returns integer as $$
BEGIN
   select 1;
END;
$$ LANGUAGE plpgsql;");

        }

        public override void Down()
        {
                Database["SqlServer"].ExecuteNonQuery(@"DROP PROCEDURE Foo");
                Database["PostgreSQL"].ExecuteNonQuery(@"DROP FUNCTION foo CASCADE");
        }
}
```

## Add Foreign Key ##

Add a foreign key constraint to the database.

_SQLite does not support foreign keys, so this is a NO OP for it_

```
using Migrator.Framework;
using System.Data;

[Migration(5)]
public class AddForeignKeyToTheBookAuthor : Migration
{
        private const string FK_NAME = "FK_Book_Author";
        public override void Up()
        {
                Database.AddForeignKey(FK_NAME, "Book", "authorId", "Author", "id");
        }
        public override void Down()
        {
                Database.RemoveForeignKey(FK_NAME);
        }
}
```

## Add A Column to an Existing Table ##

```
using Migrator.Framework;
using System.Data;

[Migration(6)]
public class AddMiddleNameToCustomer : Migration
{
        public override void Up()
        {
                Database.AddColumn("Customer", "middle_name", DbType.String, 50);
        }
        public override void Down()
        {
                Database.RemoveColumn("Customer", "middle_name");
        }
}
```

## End Run, Safety Net, Catch All ##

If you want to do something not supported, then you can use the `ExecuteNonQuery()` method to call whatever SQL you need.
In combination with the `Database["Provider"].ExecuteNonQuery()` you can still support multiple databases if need be.

```
using Migrator.Framework;
using System.Data;

[Migration(6)]
public class AddMiddleNameToCustomer : Migration
{
        public override void Up()
        {
                Database.ExecuteNonQuery("whatever SQL you want");
        }
        public override void Down()
        {
                Database.ExecuteNonQuery("whatever SQL you want");
        }
}
```