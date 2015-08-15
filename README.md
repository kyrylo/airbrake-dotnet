Airbake .NET
============

**This is a work in progress, to create a new .NET notifier library.**

<img src="http://f.cl.ly/items/0L0G1z0E2A1P3H2O042F/dotnet%2009.19.32.jpg" width=800px>

Introduction
------------

Airbrake .NET is a C# library for [Airbrake][airbrake.io], the leading exception
reporting service. Airbrake .NET provides minimalist API that enables the
ability to send _any_ .NET exception to the Airbrake dashboard. The library is
extremely lightweight, contains _no_ dependencies and perfectly suits for every
.NET application, including ASP.NET MVC, WPF, WebAPI applications.

Key features
------------

* Uses the new Airbrake JSON API (v3)
* Simple, consistent and easy-to-use library [API](#api)
* Asynchronous exception reporting (optional, enabled by default)
* Flexible logging support (configure your own logger)
* Flexible configuration options (configure as many Airbrake notifers in one
  application as you want)
* Support for proxying
* Filters support (filter out sensitive or unwated data that shouldn't be
  displayed)
* Ability to ignore certain exceptions based on their data completely (based on
  exception class, backtrace and more)
* SSL support (all communication with Airbrake is encrypted by default)
* Support for fatal exceptions (the ones that terminate your program)
* Last but not least, we follow [semantic versioning 2.0.0][semver2]

Installation
------------

### NuGet

To install [Airbrake .NET][airbrake-nuget]), run the following command in the
[Package Manager Console][pmc]):

```c#
PM> Install-Package Airbrake
```

### Manual

Download the latest
[Airbrake.dll](https://github.com/kyrylo/airbrake-dotnet/releases/latest) and
reference it in your project.

Examples
--------

### Basic example

This is the minimal example that you can use to test Airbrake .NET with your
project.

```c#
using System.Collections.Generic;
using Airbrake;

Dictionary<string, string> config;
config["projectId"] = "113743";
config["projectKey"] = "fd04e13d806a90f96614ad8e529b2822";

var airbrake = new Airbrake(config);

try
{
	int zero = 0;
	int n = 10 / zero;
}
catch (Exception ex)
{
	airbrake.Notify(ex);
}
```

Integrations with other frameworks offer _automatic_ exception reporting.

### ASP.NET MVC Applications

1. Configure Airbrake .NET inside the `Web.config` file (see the [Configuration](#configuration)
   section for available config options):

   ```c#
   <configuration>
     <configSections>
       <section name="airbrakeConfig" type="Airbrake.ConfigurationStorage.ConfigSection, Airbrake" />
     </configSections>
     <airbrakeConfig projectId="113743" />
     <airbrakeConfig projectKey="fd04e13d806a90f96614ad8e529b2822" />
   </configuration>
   ```

2. Import the `Airbrake.Clients` namespace into your application:

   ```c#
   using Airbrake.Clients;
   ```

3. Find the `App_Start > FilterConfig.cs` file and add the `WebMVCClient` error
   handler inside the `RegisterWebApiFilters` function.

   ```c#
   filters.Add(WebMVCClient.ErrorHandler());
   ```

### WPF Applications

1. Configure Airbrake .NET inside the `App.config` file (see the
   [Configuration](#configuration) section for available config options):

   ```c#
   <configuration>
     <configSections>
       <section name="airbrakeConfig" type="Airbrake.ConfigurationStorage.ConfigSection, Airbrake" />
     </configSections>
     <airbrakeConfig projectId="113743" />
     <airbrakeConfig projectKey="fd04e13d806a90f96614ad8e529b2822" />
   </configuration>
   ```

2. Import the `Airbrake.Clients` namespace into your application:

   ```c#
   using Airbrake.Clients;
   ```

3. Find the `App.xaml.cs` file and invoke `WPFClient.Start()` inside the
   `OnStartup` function.

### Web API Applications

1. Configure Airbrake .NET inside the `Web.config` file (see the
   [Configuration](#configuration) section for available config options):

   ```c#
   <configuration>
     <configSections>
       <section name="airbrakeConfig" type="Airbrake.ConfigurationStorage.ConfigSection, Airbrake" />
     </configSections>
     <airbrakeConfig projectId="113743" />
     <airbrakeConfig projectKey="fd04e13d806a90f96614ad8e529b2822" />
   </configuration>
   ```

2. Import the `Airbrake.Clients` namespace into your application:

   ```c#
   using Airbrake.Clients;
   ```

3. Find the `App_Start > FilterConfig.cs` file and add the `WebAPIClient` error
   handler inside the `RegisterWebApiFilters` function.

   ```c#
   filters.Add(WebAPIClient.ErrorHandler());
   ```

Configuration
-------------

Configration is done per-instance. That is, to start configuring, simply
instantiate the `Airbrake` class and pass the configuration `Dictionary<string,
string>`. You can create as many instances with various configurations as you
want.

```с#
Dictionary<string, string> config = new Dictionary<string, string>();
config["projectId"] = "113743";
config["projectKey"] = "fd04e13d806a90f96614ad8e529b2822";

var airbrake = new Airbrake(config);
```

In order to reconfigure the notifier, simply create a new instance of the
Airbrake class with the new options.

### Config options

#### project_id & project_key

You **must** set both `project_id` & `project_key`.

To find your `project_id` and `project_key` navigate to your project's _General
Settings_ and copy the values from the right sidebar.

![][project-idkey]

```с#
Dictionary<string, string> config = new Dictionary<string, string>();
config["projectId"] = "113743";
config["projectKey"] = "fd04e13d806a90f96614ad8e529b2822";

var airbrake = new Airbrake(config);
```

#### proxy

If your server is not able to directly reach Airbrake, you can use built-in
proxy. By default, Airbrake .NET uses direct connection.

```c#
Dictionary<string, string> config = new Dictionary<string, string>();
config["proxy"] = "john-doe:p4ssw0rd@proxy.example.com:4038";

var airbrake = new Airbrake(config);
```

#### async

By default, Airbrake .NET sends exceptions asynchronously, to minimise the
impact on your application's performance. If this is unwanted behaviour, you can
disable it.

```c#
Dictionary<string, string> config = new Dictionary<string, string>();
config["async"] = "false";

var airbrake = new Airbrake(config);
```

#### logger

By default, Airbrake .NET outputs to `STDOUT`. It's possible to add your custom
logger.

```c#
// Figure out how!
```

#### app_version

The version of your application that you can pass to differentiate exceptions
between multiple versions. It's not set by default.

```c#
Dictionary<string, string> config = new Dictionary<string, string>();
config["app_version"] = "1.0.0";

var airbrake = new Airbrake(config);
```

#### host

By default, it is set to `airbrake.io`. A `host` is a web address containing a
scheme ("http" or "https"), a host and a port. You can omit the port (80 will be
assumed) and the scheme ("https" will be assumed).

```c#
Dictionary<string, string> config = new Dictionary<string, string>();
config["host"] = "http://localhost:8080";

var airbrake = new Airbrake(config);
```

API
---

TBD.

Additional notes
----------------

### Exception limit

The maximum size of an exception is 64KB. Exceptions that exceed this limit
will be truncated to fit the size.


Supported .NET versions
-----------------------

* ?

License
-------

To be decided.

[airbrake.io]: https://airbrake.io
[airbrake-nuget]: https://www.nuget.org/packages/Airbrake/
[semver2]: http://semver.org/spec/v2.0.0.html
[notice-v3]: https://airbrake.io/docs/#create-notice-v3
[project-idkey]: https://img-fotki.yandex.ru/get/3907/98991937.1f/0_b558a_c9274e4d_orig
