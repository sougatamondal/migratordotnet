## Example ##

This setup allows you to optionally pass a version number to the migrate task. If the version number is lower than the current DB version then the `Down()` targets will be run to undo database changes. If the version number is higher than the current DB version then the `Up()` targets will be run to migrate the database to the version specified.

```
<loadtasks assembly="/Migrator.NAnt.dll" />
<target name="migrate" description="Migrate the database" depends="build">
  <property name="version" value="-1? overwrite="false" />
  <migrate
    provider="MySql|PostgreSQL|SqlServer"
    connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
    migrations="bin/MyProject.dll"
    to="${version}" />
</target>
```

To run this target

`nant migrate`

To run the migrations to a specific version you can pass the `version` property

`nant migrate -D:version=5`

## Compiling Migrations on the Fly ##

Rather than specifying a pre-compiled DLL with your migrations, you can just specify a directory that contains all the code. The migrations will then be compiled "on the fly" and executed.

_The default language to use is CSharp. If you want to use a different language pass a language parameter as well._

```
<loadtasks assembly="/Migrator.NAnt.dll" />
<target name="migrate" description="Migrate the database" depends="build">
  <property name="version" value="-1? overwrite="false" />
  <migrate
    provider="MySql|PostgreSQL|SqlServer"
    connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
    directory="migrations"
    to="${version}" />
</target>
```

## Outputing Generated SQL Statements ##

To save all of the generated SQL Statements that Migrator.Net runs against your database add a **scriptFile** attribute.

```
<loadtasks assembly="/Migrator.NAnt.dll" />
<target name="migrate" description="Migrate the database" depends="build">
  <property name="version" value="-1? overwrite="false" />
  <migrate
    provider="MySql|PostgreSQL|SqlServer"
    connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
    directory="migrations"
    to="${version}" 
    scriptFile="migrations.sql"/>
</target>
```


See the **example/example-nant.build** for a working example.