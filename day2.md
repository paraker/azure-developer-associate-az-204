# Azure App Services (PaaS like Beanstalk/Lightsail)
Requires an Aop Services Plan (think auto-scaled VM) that sets up the infra where you host your apps.<br>
App service plans comes in preconfigurations of dev/test, production and isolated.<br>
These plans have different features such as auto-scaling, redundancy etc and also capacity options.<br>
You throw your apps onto the App service plan and it deploys and deals with all the auto-scaling, backups, load-balancing, storage, cpu, ram etc.<br>
You'll get an endpoint for each app that you throw onto App Services.<br>
Note how the App Services are limited to the most popular languages out there. If you need something else, you must containerise it, or go with a traditional VM.<br>

The auto-scaling function of App Services will replicate the same containers on each app service plan instance.<br>
So typically you would probably create separate app service plans per application. Otherwise when you scale you may run containers unnecessarily.

When do we use App Services vs kubernetes or VMs etc?
Good graphic to help you [here](https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-decision-tree)

There are some configuration niceties with app services, described below.
### Configuration
You can override application settings (this is a json appSettings value for web apps) and strings in your app service.<br>
Apparently important values are `always on` and `ARR affinity`.
### Social Authentication / Authorization
You can activate to have integration with azure, microsoft, facebook, google, twitter etc.<br>
This can be enabled such that before visiting your web app you have to login.<br>
You app then needs to be able to handle the tokens given by the o(?)auth.
### SSL management
There is integrated ssl management for app services
### Clone of app
You can super simply clone your app, if you want an app for testing. This is like two clicks.
### Autoscale
Autoscaling can be performed on the app service plan.<br>
Azure will add or remove VMs based on metrics, or on date/time<br>
Metrics that can be used are CPU, memory, data in, data out, http queue, disk queue.<br>
Custom metrics can also be used with `Application Insights`.

Some example of the autoscale from azure cli examples
```
az monitor autoscale create -g {myrg} --resource {resource-id} --min-count 2 --max-count 5 \
  --count 3 --email-administrator

az monitor autoscale rule create -g {myrg} --autoscale-name {resource-name} --scale out 1 \
  --condition "Percentage CPU > 75 avg 5m"

az monitor autoscale rule create -g {myrg} --autoscale-name {resource-name} --scale in 1 \
  --condition "Percentage CPU < 25 avg 5m"
```
### Deployment Slots
Deployments slots have their own hostnames for an app within an ASP.<br>
So the app deployment slots get their own URLs.<br>
Deployment slots are used for blue-green and canary deployments.<br>
This is different to Traffic Manager using cookies to steer the traffic to Applications within the ASP.<br>

We had a pretty cool demo of a slot deployment with Azure DevOps where you can easily swap environments between staging and production.<br>
We also configured a canary deployment scenario (literally only the "traffic" field in the portal under `deployment slots` and specifying 85% and 15% for two apps in different slots)

# Traffic manager (DNS routing)
Traffic manager can sit on top of many services. It manages DNS requests. This is a free service!<br>
There are more advanced frontend app routing like Azure Front Door(name?).<br>
One good use-case for traffic manager is to route users to different regional app services.<br>
So this could be a geo-location based dns service.<br>
There are six different routing methods, four of them are described below.<br>
You can nest traffic managers so that you can stack decision making on top of each other, like first br region, second by priority etc.
### Routing methods
_Priority_,_Geographic_,_Performance_,_Weighted_<br>
* Priority prioritises one service. If the prioritised service is down, the second one will be used. Once the primary is back up that will be used again
* Geographic will route dns requests based on clients' geo-location. 
* Performance will route based on the latency to services from the furthest DNS server used by the request, or from the traffic manager. Must be the traffic manager.
* Weighted will route based on a weight, 100-1000 weighting. 

Clone a repo of a web app, then you can do the below.<br>
For example https://github.com/Azure-Samples/python-docs-hello-world
```bash
# Explcitly create resource group and app service plan
az group create --name hello-world --location centralus
az appservice plan create --name hello-world --resource-group hello-world --sku basic
az webapp create --name hello-world --resoure-group hello-world --plan hello-world

gitrepo=x
az webapp deployment source config ==name -- resource-group --repo-url --branch --manual-integration
echo

# Or super short way that creates a resource group and an app service plan for you
az webapp up --name hello-world --location centralus
```

# Azure DevOps
Microsoft's own git repo and ci/cd pipeline, kind of like gitlab<br>
Hosted in its own portal (dev.azure.com) and is free for up to 5 users.<br>
Pretty nice gui for creating its pipelines, looks very simple. <br>
Service connections is what is used to define targets for your deployments, this can be azure, aws, on-prem, whatever you want.

# Azure Function App and Functions (equivalent to aws lambda)
There is a warm-up time the first time a function is used, we're probably talking micro-seconds.<br>
For 5 minutes the function will then be cached before it is cleaned up.<br>

With an App Service Plan you can make the lambda appear in your private networks.<br>
(You'll lose the benefit of not paying for when not using it then though of course as the ASP is then active).<br>
In an App Service Plan you can also make your Function run for as long as you want of course too.<br>
Benefits of doing that would be to maintain the input bindings and not have any warm-up time.

### Hosting of Functions
Apparently you can run your functions anywhere - there should be more information about this later in the week.<br>
I think what we have to do is to install the Azure Functions Runtime on the infra that is supposed to run it.<br>
Should be able to host on Local dev machine, IoT devices, k8s, on-prem.<br>
So we saw an example of this when the instructor ran his Functions locally on his laptop in Visual Studio.<br>
No connectivity to Azure is required, we did store some analytics data in a storage account but that's it.<br>

### Trigger types for functions
There are input bindings and output bindings from a wealth of services, just like in lambda.<br>
These seemed to be able to be defined in a json file.<br>

### Durable Functions
When you need a stateful function that can live for as long as you want.<br>
Of course expensive to run this 24/7 so there an ASP is a better idea.<br>

Examples of durable functions:
* You can use durable functions to chain other functions with a special DurableOrchestrationContext syntax to call other functions.<br>
* You can also fan out (multiply to run many functions in parallel) and then fan in
* Have even human intervention in between function calls. This seems like Step Functions, but step functions are more like Azure Logic Apps.

# API Management
Helps us expose all of our APIs in our organisation in a safe and consistent manner.<br>
So we can take our old SOAP APIs, REST APIs etc and group them together in a centralised management console.<br>

Best way to import and explore your api is with a tool like `swagger` that will give you all the `operations` of your api.

* API Management uses `Transformation` so a SOAP API can be exposed as a REST API.<br>
* You can add Security (like add oauth, JWT tokens)
* You can add certificates, response headers, retry polices.
* You can limit the call rate to the APIs
* You can cache API components as well to speed up things, like a CDN
* It also geo-replicates
* and many, many more things it seems like.

Pretty cool actually how it takes the API and gets all the available `operations`.<br>
Like GET `GetSession`, `GetSessions`, `GetTopic` or perhaps PUT `this` or `that`.<br>
For each of these operations, or for the whole API, or for the whole grouping `product` you can set policies.<br>
These policies are what's described above in the bullet list.<br>
They can be applied at the frontend, inbound, outbound or at the backend. Really cool. Best for understanding is to look a the portal GUI<br>
The policies can be written in XML too, or just done in the Azure Portal.

A `product` is just a logical grouping of APIs, for easy management.<br>
Each product can have multiple APIs in them, obviously. Perhaps you have a couple of APIs being paid APIs and some APIs are free. But it is still the same API. These can be grouped together.

An example that we walked through was to remove the application version from an api, to hide what language and version the API was written in.<br>
This was available in the header, and we configured API management to remove the header `x-aspnet-version`<br>. This was done in XML code by using some syntax.
This was a policy applied on the outbound level, as it was a response header.<br>

You can override the product or API level policies per operation if you want. <br>
It seemed like a pretty understandable order of operation in the XML, basically there was a baseline that you can put policies before or after.<br>
There is also a GUI generator for the XML.

The `<base />` element inherits all policies from the corresponding section.<br>
Remove this block to prevent inheritance for this particular policy.
```xml
<outbound>
    <base />
    <xml-to-json kind="direct" apply="always" consider-accept-header="false" />
</outbound>
```

Pretty cool here with the API Management can also show you like in-depth tracing of each of the steps along the way.<br>
This is probably somewhat similar to what the appdynamics or datadog can do perhaps? Not sure!