## A Simple Example ##

This example just runs the migrations from the current DB version to the highest migration.

```
<PropertyGroup>
    <MigratorTasksPath>$(MSBuildProjectDirectory)\migrator</MigratorTasksPath>
</PropertyGroup>
    
<Import Project="$(MigratorTasksPath)\Migrator.Targets" />

<Target name="Migrate" DependsOnTargets="Build">
    <Migrate Provider="SqlServer"
        Connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
        Migrations="bin/MyProject.dll"/>
</Target>
```

Run the migration with MSBuild to update the database schema to the latest version

e.g. `MSBuild build.proj /t:Migrate`

## More Complex Example ##

If you need a bit more flexibility you can also setup your task to allow you to optionally pass a version number to it. If the version number is lower than the current DB version then the `Down()` targets will be run to undo database changes. If the version number is higher than the current DB version then the `Up()` targets will be run to migrate the database to the version specified.

In this case we've called the property `SchemaVersion`.

```
<Target name="Migrate" DependsOnTargets="Build">
    <CreateProperty Value="-1"  Condition="'$(SchemaVersion)'==''">
        <Output TaskParameter="Value" PropertyName="SchemaVersion"/>
    </CreateProperty>
    <Migrate Provider="SqlServer"
            Connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
            Migrations="bin/MyProject.dll"
            To="$(SchemaVersion)"/>
</Target>
```

Run the migration with MSBuild to a specific version (either up or down)

`MSBuild build.proj /t:Migrate /p:SchemaVersion=5`

Run the migrations to bring the database to the latest version just don't pass a `SchemaVersion` property

`MSBuild build.proj /t:Migrate`

## Compiling Migrations on the Fly ##

Rather than specifying a pre-compiled DLL with your migrations, you can just specify a directory that contains all the code. The migrations will then be compiled "on the fly" and executed.

_The default language to use is CSharp. If you want to use a different language pass a Language parameter as well._

```
<Target name="Migrate" DependsOnTargets="Build">
    <CreateProperty Value="-1"  Condition="'$(SchemaVersion)'==''">
        <Output TaskParameter="Value" PropertyName="SchemaVersion"/>
    </CreateProperty>
    <Migrate Provider="SqlServer"
            Connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
            Directory="migrations"
            To="$(SchemaVersion)"/>
</Target>
```

## Outputing Generated SQL Statements ##

To save all of the generated SQL Statements that Migrator.Net runs against your database add a **ScriptFile** attribute.

```
<Target name="Migrate" DependsOnTargets="Build">
    <CreateProperty Value="-1"  Condition="'$(SchemaVersion)'==''">
        <Output TaskParameter="Value" PropertyName="SchemaVersion"/>
    </CreateProperty>
    <Migrate Provider="SqlServer"
            Connectionstring="Database=MyDB;Data Source=localhost;User Id=;Password=;"
            Directory="migrations"
            To="$(SchemaVersion)"
            Scriptfile="migrations.sql"/>
</Target>
```

See the **example/example-msbuild.proj** for a working example.

## Full Example ##

Here's a full MSBuild example that compiles and runs the migrations.

```
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <PropertyGroup>
        <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
        <ClassLibraryOutputDirectory>bin\$(Configuration)</ClassLibraryOutputDirectory>
        <MigratorTasksPath>$(MSBuildProjectDirectory)\migrator</MigratorTasksPath>
        <MigrationsProject>ProjectMigrations\ProjectMigrations.csproj</MigrationsProject>
    </PropertyGroup>
    
    <Import Project="$(MigratorTasksPath)\Migrator.Targets" />
    
    <Target Name="Build-Migrations">
        <MSBuild Projects="$(MigrationsProject)" Targets="Build">
            <Output TaskParameter="TargetOutputs" ItemName="MigrationAssemblies" />
        </MSBuild>
        
        <Message Text="Built: @(MigrationAssemblies)"/>
    </Target>
    
    <Target Name="Migrate" DependsOnTargets="Build-Migrations">
        <Message Text="Migrating: @(MigrationAssemblies)"/>
        
        <CreateProperty Value="-1"  Condition="'$(SchemaVersion)'==''">
            <Output TaskParameter="Value" PropertyName="SchemaVersion"/>
        </CreateProperty>
        <Migrate Provider="SqlServer"
            Connectionstring="Database=test2;Data Source=localhost;User Id=sa;Password=sql;"
            Migrations="@(MigrationAssemblies)"
            To="$(SchemaVersion)"/>
    </Target>
</Project>
```