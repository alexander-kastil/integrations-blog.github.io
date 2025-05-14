+++
date = '2023-02-05T13:22:53+02:00'
title = 'Use Dapr with Azure Container Apps'
tags = ['Cloud Native', 'Azure', 'Dapr']
+++

In this post, I will show you how to use Dapr with Azure Container Apps (ACA) and demonstrate how easy it is to use Dapr for Lift and Shift scenarios. The complete code and details can be found in my [az-204 GitHub repository](https://github.com/alexander-kastil/az-204/tree/main/demos/05-containers/demo-08).

Dapr is a portable, event-driven runtime that makes it easy for developers to build resilient, microservices-based applications. It provides a set of building blocks for common application patterns, such as service invocation, state management, and pub/sub messaging. Dapr can be used with any programming language and framework, and it can run on any cloud or on-premises environment.

![dapr-architecture](/images/posts/aca-dapr/dapr-architecture.png)

Our sample consists of a simple .NET 9 Web API that uses Dapr to save its state, a simple count. It includes the Dapr.Client and Dapr.AspNetCore NuGet packages. The API exposes a single endpoint that increments the count and returns the current value. The API uses Dapr's state management building block to store the count in a Redis state store. A full list of Dapr state stores can be found in the [Dapr State stores documentation](https://docs.dapr.io/reference/components-reference/supported-state-stores/).

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: food-state
spec:
  type: state.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: ""
```

In order to use Redis locally, you need to install it. You can do this using Docker:

```bash
docker run -d --name redis/redis-stack -p 6379:6379 redis
```

The DaprClient is registered in Program.cs:

```csharp
builder.Services.AddDaprClient();
```

The DaprClient is injected into the controller and the name of the state store is kept in the appsettings.json file as `DAPR_STATE_STORE`. The `GetStateAsync` and `SaveStateAsync` methods are used to get and save the state, respectively.

```c#
[Route("[controller]")]
[ApiController]
public class CountController : ControllerBase
{
    string storeName = "";
    const string key = "counter";

    private readonly DaprClient client;

    public CountController(DaprClient daprClient, IConfiguration cfg)
    {
        client = daprClient;
        storeName = cfg.GetValue<string>("DAPR_STATE_STORE");
    }

    [HttpGet("get")]
    public async Task<int> Get()
    {
        var counter = await client.GetStateAsync<int>(storeName, key);
        await client.SaveStateAsync(storeName, key, counter + 1);
        return counter;
    }
}
```

The App will be deployed to Azure Container Apps into an existing environment. Just for demonstration purposes, we will switch the Redis state store to use an Azure Storage Account and create an alternative component file:

```yaml
componentType: state.azure.blobstorage
version: v1
metadata:
  - name: accountName
    value: "foodstoragedev"
  - name: accountKey
    value: "<ACCOUNT_KEY>"
  - name: containerName
    value: "food-state"
```

In the Container App Environment, we will create a new component using the Azure CLI. The component name is `food-state` and the type is `state.azure.blobstorage`.

```bash
az containerapp env dapr-component set -n $acaenv -g $grp \
    --dapr-component-name $statestore \
    --yaml statestore-blob.yaml

```

Deployment of the container app will be done using the Azure CLI. Note that .NET 8+ is using `8080` as the default port.

```bash
pwd=$(az acr credential show -n $acr -g $grp --query passwords[0].value -o tsv)
loginSrv=$(az acr list -g $grp --query "[?name=='$acr'].loginServer" -o tsv)

...

az containerapp create -n $app -g $grp \
    --image $img \
    --environment $acaenv \
    --target-port 8080 --ingress external \
    --min-replicas 1 --max-replicas 1 \
    --enable-dapr \
    --dapr-app-port 8080 \
    --dapr-app-id $app \
    --registry-server $loginSrv \
    --registry-username $acr \
    --registry-password $pwd
```
