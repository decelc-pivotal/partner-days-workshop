---
pageTitle: Set up
---

# Introduction

In this exercise, we'll set up our workstation and cloud environment so 
that we're ready to build and run modern .NET applications.

__Your instructor will provide the url and credentials for the Pivotal
Cloud Foundry (PCF) instance you will be using.__

Alternatively, you can sign up for a trial account of the hosted version
of PCF, called Pivotal Web Services:

1. Go to [http://run.pivotal.io](http://run.pivotal.io) and choose 
"sign up for free."

1. Click "create account" link on sign up page.

1. Fill in details.

1. Go to email account provided and click on verification email link.

1. Click on "claim free trial" link and provide phone number.

1. Validate your account and create your organization.

# Package manager

We suggest using a package manager to install bootcamp software.

-   __MacOS:__

    [Homebrew](https://brew.sh/) (`brew search PACKAGE` to search)

-   __Windows:__

    [Chocolatey](https://chocolatey.org/) (`choco search PACKAGE` to
    search)

-   __Debian-Based Linux:__

    [Apt](https://wiki.debian.org/Apt) (`apt search PACKAGE` to search)
    
-   __Fedora-Based Linux:__

    [Yum](http://yum.baseurl.org/) (`yum search PACKAGE` to search)

# Install Cloud Foundry CLI

You can interact with Cloud Foundry via Dashboard, REST API, or 
command line interface (CLI). Here, we install the CLI and ensure it's 
configured correctly.

-   __Windows:__
    ```bash
    choco install cloudfoundry-cli
    ```

-   __MacOS:__
    ```bash
    brew install cloudfoundry/tap/cf-cli
    ```

-   __Debian and Ubuntu:__
    ```bash
    wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
    echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
    sudo apt-get update
    sudo apt-get install cf-cli
    ```

-   __RHEL, CentOS, and Fedora:__
    ```bash
    sudo wget -O /etc/yum.repos.d/cloudfoundry-cli.repo https://packages.cloudfoundry.org/fedora/cloudfoundry-cli.repo
    sudo yum install cf-cli
    ```

Confirm that it installed successfully by going to a command line,
and typing `cf -v`


# Install .NET Core and Visual Studio Code

.NET Core represents a modern way to build .NET apps, and here we make 
sure we have everything needed to build ASP.NET Core apps.

1. Visit [https://www.microsoft.com/net/download](https://www.microsoft.com/net/download)
 to download and install the latest version of the .NET Core SDK.
 
1. Confirm that it installed correctly by opening a command line and 
typing `dotnet --version`

1. Install [Visual Studio Code](https://code.visualstudio.com) using
your favorite package manager.

1. Open Visual Studio Code and go to __View → Extensions__

1. Search for `C#` and choose the top `C# for Visual Studio Code` option 
and click `Install.` This gives you type-ahead support for C#.


# Create an ASP.NET Core project

Visual Studio Code makes it easy to build new ASP.NET Core projects. 
We'll create a sample project just to prove we can!

1. Within Visual Studio Code, go to __View → Integrated Terminal__. The 
Terminal gives you a shell interface without leaving Visual Studio Code.

1. Navigate to a location where you'll store your project files 
(e.g. C:\BootcampLabs) and create a sub-directory called "mvctest" inside.

1. Navigate into the newly created "mvctest" directory and type in 
`dotnet new mvc` to create a new ASP.NET Core MVC project.

1. In Visual Studio Code, click __File → Open__ and navigate to the 
directory containing the new ASP.NET Core project.

1. Observe the files that were automatically generated. Re-open the 
Terminal window.

1. Start the project by typing `dotnet run` and visiting 
[http://localhost:5000](http://localhost:5000). To stop the
application, enter `Ctrl+C`.

# Deploy ASP.NET Core application to Cloud Foundry

Let's push an app! Here we'll experiment with sending an application to Cloud
Foundry.

1. In Visual Studio Code, go to __View → Extensions__.

1. Search for "Cloudfoundry" and install 
`Cloudfoundry Manifest YML support` extension. This gives type-ahead 
support for Cloud Foundry manifest files.

1. In Visual Studio Code, create a new file called manifest.yml at base 
of your project.

1. Open the `manifest.yml` file, and type in the following (notice the 
typing assistance from the extension):

    ```yaml
    ---
    applications:
    - name: core-cf-[enter your name]
      buildpack: https://github.com/cloudfoundry/dotnet-core-buildpack#v2.1.4
      instances: 1
      memory: 256M
    ```

1. In the Terminal, type in `cf login -a <PCF API url>` and provide
your credentials. Now you are connected to Pivotal Cloud Foundry.

1. Enter `cf push` into the Terminal, and watch your application get 
bundled up and deploy to Cloud Foundry.

1. In Pivotal Cloud Foundry Apps Manager, see your app show up, andvisit the app’s URL.


# Instantiate Spring Cloud Services instances

Spring Cloud Services wrap up key Spring Cloud projects with managed capabilities.
Here we create a pair of these managed services.

1. In Pivotal Cloud Foundry Apps Manager, click on your "space" on the 
left, and switch to the "Services" tab. Note that all of these 
activities can also be done via the CF CLI.

1. Click "Add Service."

1. Type "Spring" into the search box to narrow down the choices.

1. Select "Service Registry" and select the default plan.

1. Provide an instance name and do not choose to bind the service to 
any existing applications. Click "Create." This service will take a
couple of minutes to become available.

1. Repeat step 3 above and choose "Config Server" from the marketplace.

1. Choose the default plan, provide an instance name, and click
"Create." Wait a couple minutes before expecting to see this service
fully operational.

1. Return to your default space in Apps Manager, click "Services",
choose Service Registry, and click the "manage" link. This takes you to 
the Eureka dashboard.

1. Return again to the default space, click "Service", choose Config
Server, and click the "manage" link. Nothing here just yet!
