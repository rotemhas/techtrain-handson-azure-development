

# Event streaming  - malicious ip address in real time.
lets add an eventhub consumer to out functions app and consume the events from the apim. for every incoming ip address we will check if it is malicious and if so we will create an alert.

1. Get supporting files:
    1. classes:
        * VirusTotalClient
        * src/backend/Models/*.cs
    1. dependencies:
        - Microsoft.Extensions.Caching.Memory -
        - Microsoft.Extensions.Http -
    1. startup.cs
        ```csharp
        .AddHttpClient()
        .AddMemoryCache()
        ```

1. add defaultazurecredentials to backend function app:
    ```bash
    dotnet add package Azure.Identity --version 1.10.4
    ```
    1. add singleton DefaultAzureCredentials to startup.cs with includeInteractiveCredentials: true
    ```csharp
    .AddSingleton<DefaultAzureCredential>(_ => new DefaultAzureCredential(includeInteractiveCredentials: true))
    ```

1. add eventhub trigger for real time processing of requests from the apim logger event hub:
    1. Create a dedicated consumer group for backend function app in order to consume events from the eventhub without interfering others.
    1. update backend function app with the following configurations, in both local.settings.json and main.bicep:
        ```json
        EventHubRequestsName: <eventhub-name>
        EventHubRequestsConnectionOptions__fullyQualifiedNamespace: <eventhub-endpoint>
        EventHubRequestsConsumerGroup: <eventhub-consumer-group>
        ```
    1. Create new eventhubtrigger
        ```bash
        func new --template "EventHubTrigger" --name "EventHubRequestTrigger"
        ```
    1. update new eventhub trigger with the eventhub configurations.
    1. test function locally by simulating requests to the apim.



1. extract ip from payload
    1. newtonsoft


1. VirusTotal
    1. Create a free account in https://www.virustotal.com/gui/join-us.
    1. Locate your virus total api key under your profile and copy it.
        <img src="../imgs/virustotal-apikey.png"/>
    1. Place the api key in the key vault:
        ```bash
        # Set the following variables
        keyVaultName=<your-key-vault-name> # Can be found in .env/AZURE_KEY_VAULT_NAME
        secretName="VIRUSTOTAL-API-KEY"
        virusTotalApiKey=<your-virustotal-api-key>

        # Retrieve your user object id from Azure Entra ID
        objectId=$(az ad signed-in-user show --query id -o tsv)

        # Create a Key Vault Access Policy with secret set permissions
        az keyvault set-policy --name $keyVaultName --secret-permissions set --object-id $objectId

        # Create new secret in the keyvault
        az keyvault secret set --vault-name $keyVaultName --name $secretName --value $virusTotalApiKey

        # Remove the Key Vault Access Policy you created earlier
        az keyvault delete-policy --name $keyVaultName --object-id $objectId
        ```

1. virus total integration:
    1. Initialize VirusTotalClient using the api-key from key vault using managed identity
        1. add AZURE_KEY_VAULT_ENDPOINT with your key vault endpoint to local.settings.json
        <img src="../imgs/localsettings-keyvault-endpoint.png"/>
    1. add dependencies for reading keyvault secrets
        ```bash
        dotnet add package Azure.Security.KeyVault.Secrets --version 4.5.0
        ```

        now we will be able to connect to azure services using managed identity from within the devcontainer.
    1. Create VirusTotalClient singleton and initiate it with the api key from the key vault. Since the application will have now dependency on the key vault and virus total, it will be wise to asses that VirusTotalClient is healthy while the application is running. You can leverage Healthz endpoint for that.


        > Note: VirusTotalClient wraps a restful virustotal api, so we must provide him an httpclient. to reduce the number of calls we can provide him a MemoryCache.
    1. In order to run the functions locally grant yourself with Secrets Get permission on the key vault.
        ```bash
        # Set the following variables
        keyVaultName=<your-key-vault-name> # Can be found in .env/AZURE_KEY_VAULT_NAME

        # Retrieve your user object id from Azure Entra ID
        objectId=$(az ad signed-in-user show --query id -o tsv)

        # Create a Key Vault Access Policy with secret set permissions
        az keyvault set-policy --name $keyVaultName --secret-permissions get --object-id $objectId
        ```

        > Note: since this permission is not part of the IaC, you will need to grant yourself with this permission every time after a deployment in order to run the function locally.

1. Persist data in cosmos db
    1. create new table in cosmos db.
    1. add cosmos configurations to bicep and local.settings.json:
        1. CosmosDatabaseName
        1. CosmosConnectionOptions__accountEndpoint

1. add eventhub trigger



# cosmos trigger
problem: we want to have a timer for each task that has an overdue date configured, and we want to change the status of the task to overdue when the timer is triggered.

“Durable entities” is a feature in Azure Functions that enables you to build stateful functions in a serverless environment. Durable entities are similar to virtual actors or grains, as popularized by the Orleans project. They are addressable via an entity ID and execute operations serially, one at a time, to prevent race conditions. Durable entities are created implicitly when they’re called or signaled.

The “actor model pattern” is a design pattern for concurrent computation that treats “actors” as the universal primitives of concurrent computation 2. In this pattern, actors are independent units of computation that communicate with each other by exchanging messages 2. Each actor has its own state and can perform operations when prompted to do so 2. The actor model pattern is often used in distributed systems, where it is important to ensure that different parts of the system can operate independently and without interfering with each other 2.

Durable entities and the actor model pattern are related in that they both provide a way to build stateful functions that can preserve state and data. Durable entities are similar to virtual actors in that they are addressable via an entity ID and execute operations serially, one at a time, to prevent race conditions. Both durable entities and the actor model pattern provide a means for scaling out applications by distributing the work across many entities, each with a modestly sized state.

For more information on durable entities in Azure Functions, see Durable entities - Azure Functions1. For more information on the actor model pattern, see Actor model - Wikipedia 2.
https://azure.microsoft.com/en-us/updates/durable-functions-now-supports-a-new-stateful-entities-pattern/


1. update your principalId in main.parameters.json to have data plane permissions on keyvault and cosmos db
       # Retrieve your user object id from Azure Entra ID
        objectId=$(az ad signed-in-user show --query id -o tsv)

```bash
azd env set AZURE_PRINCIPAL_ID $(az ad signed-in-user show --query id -o tsv)
```

1. Serverless event-based architectures with Azure Cosmos DB and Azure Functions
Requirements
To implement a serverless event-based flow, you need:

The monitored container: The monitored container is the Azure Cosmos DB container being monitored, and it stores the data from which the change feed is generated. Any inserts, updates to the monitored container are reflected in the change feed of the container.
The lease container: The lease container maintains state across multiple and dynamic serverless Azure Function instances and enables dynamic scaling. You can create the lease container automatically with the Azure Functions trigger for Azure Cosmos DB. You can also create the lease container manually. To automatically create the lease container, set the CreateLeaseContainerIfNotExists flag in the configuration. Partitioned lease containers are required to have a /id partition key definition.
=> create leases container

1.
    ```bash
    pushd src/backend
    dotnet add package Microsoft.Azure.WebJobs.Extensions.DurableTask --version 2.11.1

    # i could use httptrigger and use cosmos output trigger instead
    dotnet add package Microsoft.Azure.Cosmos --version 3.35.3
    popd


    1. create a new CosmosTodoItemTrigger class with a function that will be triggered by cosmos db change feed.
    for every created/updated TodoItem, update the the status of the task if due date is passed or create a new timer for the due date.

    ```bash
    func new --template "Cosmos DB trigger" --name "CosmosTodoItemTrigger"
    ```

    update hosts.json
    ```json
        "extensions": {
            "durableTask": {
                "hubName": "Todo"
            }
        }
    ```


azd deploy backend

recommended course - Building Event-driven Microservices with the Azure Cosmos DB Change Feed
https://app.pluralsight.com/library/courses/building-event-driven-microservices-azure-cosmos-db-change-feed/table-of-contents

# caching

# UI

# Troubleshooting
* Functions not working as expected? try to change the log level to verbose:
    ```json
    {
        "logging": {
            "logLevel": {
                "default": "Information"
            }
        }
    }
    ```




# Additional resource
| Name | Description |
| --- | --- |
| Use dependency injection in .NET Azure Functions  | https://learn.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection |
| Monitor the health of App Service instances - Azure App Service | https://learn.microsoft.com/en-us/azure/app-service/monitor-instances-health-check?tabs=dotnet#enable-health-check