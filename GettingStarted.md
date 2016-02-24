# Introduction #

MigratorDotNet is a database versioning system much like Ruby on Rail's Migrations.  Its purpose is to automate your database changes, and help keep those changes in sync throughout your environments.

## Supported Databases ##

  * MySQL (5.0, 5.1)
  * Oracle (not well tested?)
  * PostgreSQL
  * SQLite
  * SQL Server (2000, 2005, CE)

## Supported Execution Modes ##
  * MSBuildTarget
  * NAntTask
  * Console Application (You should be using an automation tool! :))

## Overview ##

The basic concept here is that you create an assembly that contains your "Migrations". The Migrations are classes that derive from the Migration class. Upon deriving from Migration you will need to write the code necessary to "Up" that migration or "Down" that migration. When you migrate to that version of the database, the Up() method will be called and executing whatever database modifications need to be ran in order to bring your database to that version. When you migrate in reverse to a previous version, the Down() method will be called executing database modifications in order to undo that set of database changes.

Each Migration should be a very small incremental step in the database. Common sizes of Migrations are creating a table and adding one or more columns to a table, altering the data of a table, running a custom query with ExecuteQuery on tables. Keep your migrations as small as possible as this will facilitate easier steps between versions, much like you would in a traditional version control system such as VSS or SVN.

A sample migration would be as follows:

```
using Migrator.Framework;
using System.Data;

namespace DBMigration
{
	[Migration(20080401110402)]
	public class CreateUserTable_001 : Migration
	{
		public void Up()
		{
			Database.CreateTable("User",
				new Column("UserId", DbType.Int32, ColumnProperties.PrimaryKeyWithIdentity),
				new Column("Username", DbType.AnsiString, 25)
				);
		}

		public void Down()
		{
			Database.RemoveTable("User");
		}
	}
}
```
Where the Migration Attribute indicates the order (or in version 0.8 and greater) the timestamp of this migration.


## Installation and Configuration With Visual Studio ##

Here are the steps to add migrations to an existing solution in Visual Studio:
  1. Add a new class library project.  In this example it's called Called DBMigration.
  1. Create a lib directory in your DBMigration project, and extract all the dotnetmigrator DLLs into it. You can exclude database-specific DLLs that you don't need.  e.g. If you're not using Oracle, you don't need Oracle.DataAccess.dll.
  1. In your DBMigration project, add a reference to the Migrator.Framework.dll. That's the only additional reference you need in the project.
  1. create a build.proj file in your DBMigration project dir.  Refer to the full example in **MSBuildTarget.wiki**.  If you're using SQL Server, the only attribute you should need to adjust is the Connectionstring.
  1. Create your first migration.
> > e.g. File Name: CreateUser\_001.cs
> > > Class Name: CreateUser\_001
  1. Compile and run your Migrations. The easiest way to run your migrations is to create two External Tools references for your migrations targets - one for going up, and one for going down to a specific version.  Here's what the configurations look like if you're using MSBuild with .NET 2.0:


> Title: Migrate Up
> Command: C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\MSBuild.exe
> Arguments: build.proj /t:migrate
> Initial Directory: $(ProjectDir)
> Check the "Use Output window" box.

> Title: Migrate To Version
> Command: C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\MSBuild.exe
> Arguments: build.proj /t:migrate
> Initial Directory: $(ProjectDir)
> Check the "Use Output window" box
> Also check the "Prompt for arguments" box.  You'll use that to specify which version you want to go to. (e.g. 0 for all the way back down).


  * For more details on the migrator tasks available, see WritingMigrations.