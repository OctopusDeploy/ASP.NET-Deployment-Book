# Configuring Applications

Long-time .NET developers will be familiar with configuration files like `web.config` and `app.config`, as well as web configuration transform files (`web.release.config`). Applications read configuration from these files using the `System.Configuration` assembly. 

In ASP.NET 5 and DNX, the configuration model has changed. Instead of the `.config` file format, you can provide configuration data from a variety of sources:

 - In code
 - JSON files
 - XML files
 - INI files
 - Environment variables
 - Command line arguments

At runtime, 

