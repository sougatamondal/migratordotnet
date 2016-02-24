# Introduction #

The primary mechanism for building and testing the project is a NAnt build taks in default.build. The fact that it works in Visual Studio is nice, but the NAnt tasks need to be maintained as the primary build. This allows us to support both Microsoft .NET and Mono at the same time.

To override any of the build properties copy local.properties-exmple to local.properties and override any of the
property values in the default.build.

The primary things to override are:
  * Tests that you need to exclude because you don't have the proper databases to test all of the Providers
  * The location of your config file that contains the ConnectionStrings to use for each of the databases

Example: local.properties
```
<project xmlns="http://nant.sf.net/release/0.85/nant.xsd">
	<property name="tests.exclude" value="Oracle,SqlServer,Postgre"/>
        <property name="tests.app.config" value="${dir.config}/local.config"/>
</project>
```

# Windows #

NAnt is included with the code and there is a helper build.bat file that references the included NAnt.

To build the project on Windows simply run the build.bat file in this folder like so:

```
c:\> build
```

This will run the nant build in default configuration. You can pass a target to the build.bat to run a specific
target in the default.build file.

To zip the project into a zip file run build passing a zip argument like so:

```
c:\> build zip
```