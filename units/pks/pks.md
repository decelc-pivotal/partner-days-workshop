---
pageTitle: Exercise #2 - Working with a Configuration Store
---

# Introduction 

In this exercise, we get hands-on with a Git-backed config store and build an app that
consumes it.


# Point Config Server to Git repository

1. Create a file called `config.json` with the following contents. We
will use this file to tell the Config Server where to get its 
configurations from.

    ```json
    {
      "git": {
        "uri": "https://github.com/platform-acceleration-lab/cloud-native-net-configs"
      }
    }
    ```
    
1. From the Terminal within Visual Studio Code, enter in the following 
command.
 
    ```bash
    cf update-service <your config server name> -c config.json
    ```
    
1. From Apps Manager, navigate to the "Services" view, manage the Config
Server instance, and notice the git URL in the configuration.


# Create ASP.NET Core Web API

Here we create a brand new microservice and set it up with the Steeltoe 
libraries that pull configuration values from our Spring Cloud Services 
Config Server.

1. In the Visual Studio Code Terminal window, navigate up one level and 
create a new folder (bootcamp-webapi) to hold our data microservice.

1. From within that folder, run the 
`dotnet new webapi` command in the
Terminal. This uses the "web api" template to create scaffolding for a 
web service.

1. From the Terminal enter

    ```bash
    dotnet add package Microsoft.Extensions.Options
    dotnet add package Microsoft.Extensions.Configuration
    dotnet add package Pivotal.Extensions.Configuration.ConfigServerCore --version 2.1.0
    ```
    
1. Open the newly created folder in Visual Studio Code.

1. In the Controllers folder, create a new file named 
`ProductsController.cs`.

1. Enter the following code. It defines a new API controller, specifies 
the route that handles, it, and an operation we can invoke.

    ```csharp
    using System;
    using System.Collections.Generic;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Configuration;
    
    namespace bootcamp_webapi.Controllers
    {
        [Route("api/[controller]")]
        public class ProductsController : Controller
        {
            private readonly IConfigurationRoot _config;
    
            public ProductsController(IConfigurationRoot config)
            {
                _config = config;
            }
    
            // GET api/products
            [HttpGet]
            public IEnumerable<string> Get()
            {
                Console.WriteLine($"connection string is {_config["productdbconnstring"]}");
                Console.WriteLine($"Log level from config is {_config["loglevel"]}");
                return new[] {"product1", "product2"};
            }
        }
    }
    ```
    
# Connect application to Config Server

Next, we add what's needed to make our ASP.NET Core application 
retrieve it's configuration from Cloud Foundry and the Config Server. 

1. Edit `appsettings.json` to include the application name and cloud 
    config name. This maps to the configuration file read from the server.

    ```diff
    {
      "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
          "Default": "Warning"
        }
      },
    + "spring": {
    +   "application": {
    +     "name": "branch1"
    +   },
    +   // determines the name of the files pulled; explicitly set to avoid
    +   // env variable overwriting it
    +   "cloud": {
    +     "config": {
    +       "name": "branch1"
    +     }
    +   }
    + }
    }
    ```

1. In `Program.cs`, set up configuration providers in the following 
order:

    1. Environment variables
    1. `appsettings.json`
    1. Environment-specific `appsettings.json`
    1. Our config store 

    ```diff
    // ...
    + using Pivotal.Extensions.Configuration.ConfigServer;
    
    namespace bootcamp_webapi
    {
        public class Program
        {
            // ...
            
            public class Program
            {
                public static void Main(string[] args)
                {
                    CreateWebHostBuilder(args).Build().Run();
                }

                public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
                    WebHost.CreateDefaultBuilder(args)
    +                   .UseCloudFoundryHosting()
    +                   .ConfigureAppConfiguration(b => b.AddConfigServer(new LoggerFactory().AddConsole(LogLevel.Trace)))
                        .UseStartup<Startup>();
            }
        }
    }
    ```
    
1. In `Startup.cs`, add configuration to the service container, allowing
it to be injected into the `ProductsController`.

    ```diff
      // ...
    + using Pivotal.Extensions.Configuration.ConfigServer;
    
      namespace bootcamp_webapi
      {
          public class Startup
          {
                // ...

                public void ConfigureServices(IServiceCollection services)
                {
                    // ...

    +               services.AddConfiguration(Configuration);

                    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
                }

                // ...
          }
      }

    ```
    
1. Add a `manifest.yml` file to the base of the project. This tells 
Cloud Foundry how to deploy your app. Enter:

    ```yaml
    ---
    applications:
    - name: core-cf-microservice-<enter your name>
      buildpack: https://github.com/cloudfoundry/dotnet-core-buildpack#v2.0.5
      instances: 1
      memory: 256M
      # determines which environment to pull configs from
      env:
        ASPNETCORE_ENVIRONMENT: dev
      services:
        - <your config server instance name>
    ```
    
1. Execute `cf push` to deploy this application to Cloud Foundry! Note
the __route__ of your microservice:

    ```no-highlight
    Waiting for app to start...
    
    name:              core-cf-yourname
    requested state:   started
    instances:         1/1
    usage:             256M x 1 instances
    routes:            core-cf-yourname.apps.chicken.pal.pivotal.io
    last uploaded:     Wed 28 Mar 09:19:42 MDT 2018
    stack:             cflinuxfs2
    buildpack:         https://github.com/cloudfoundry/dotnet-core-buildpack#v2.0.5
    start command:     cd ${DEPS_DIR}/0/dotnet_publish && ./mvctest --server.urls http://0.0.0.0:${PORT}
    ```

# Observe behavior when changing application name or label

See how configurations are pulled and used.

1. Navigate to the `/api/products` endpoint of your microservice and
seeing a result.

1. Go to the "Logs" view in Apps Manager and see the connection string 
and log level written out.

1. Go back to the source code and change the application name and cloud 
config name in the `appsettings.json` to "branch3", and in the 
`manifest.yml` change the environment to "qa." This should resolve to a 
different configuration file in the GitHub repo, and load different 
values into the app's configuration properties.

1. Re-deploy the app (`cf push`), hit the API endpoint, and observe the 
different values logged out.
