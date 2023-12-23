# Workshop: Monitor App Service instances using Health check

- [Introduction](#introduction)
- [Learning Objectives](#learning-objectives)
- [Challenges](#challenges)
    - [Challenge 1: Expose Healthz endpoint in a web server](#challenge-1)
    - [Challenge 2: Configure health check probe for App Service instance](#challenge-2)
- [Additional Resources](#additional-resources)

## Learning Objectives <a name="learning-objectives"></a>
1. tbd

## Challenges <a name="challenges"></a>
1. Expose Healthz endpoint in a web server.
1. Configure health check probe for App Service instance.


### Challenge 1: Expose Healthz endpoint in a web server <a name="challenge-1"></a>

### Challenge 2: Configure health check probe for App Service instance <a name="challenge-2"></a>

1. under backend.bicep add `healthCheckPath: '/api/healthz'` to the function app resource.
1. validate function is healthy



# Additional resource
| Name | Description |
| --- | --- |
| Monitor App Service instances using Health check | https://learn.microsoft.com/en-us/azure/app-service/monitor-instances-health-check?tabs=dotnet |