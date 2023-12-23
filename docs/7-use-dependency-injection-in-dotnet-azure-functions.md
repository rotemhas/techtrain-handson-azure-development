# Workshop: Use dependency injection in .NET Azure Functions

- [Introduction](#introduction)
- [Learning Objectives](#learning-objectives)
- [Challenges](#challenges)
    - [Challenge 1: Update a function app to use dependency injection](#challenge-1)
- [Additional Resources](#additional-resources)

## Introduction <a name="introduction"></a>
tbd

## Learning Objectives <a name="learning-objectives"></a>
tbd

## Challenges <a name="challenges"></a>
1. Bootstrapping and provisioning an initial template for your Azure environment.
1. Configure CI/CD using Github Actions.

### Challenge 1: Update a function app to use dependency injection<a name="challenge-1"></a>

1. add dependencies
    ```bash
    pushd src/backend
    dotnet add package Microsoft.Extensions.DependencyInjection --version 7.0.0
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    popd
    ```

1. create new file Startup.cs:
    ```csharp
    using Microsoft.Azure.Functions.Extensions.DependencyInjection;
    using Microsoft.Extensions.DependencyInjection;

    [assembly: FunctionsStartup(typeof(backend.Startup))]

    namespace backend
    {
        public class Startup : FunctionsStartup
        {
            public override void Configure(IFunctionsHostBuilder builder)
            {

                builder.Services.AddSingleton<Healthz>();
            }
        }
    }
    ```
1. update Healthz.cs function to be non-static
1. validate app is running locally


# Additional resource
| Name | Description |
| --- | --- |
| Use dependency injection in .NET Azure Functions  | https://learn.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection |
