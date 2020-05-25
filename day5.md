## Event Grid (Equivalent to AWS SNS)
`Event Producers` generate notifications that are passed to the `Event Ingestion` in `Event Grid`.<br>
Then `Event Consumers` can consume this data (optional).<br>
Loads of services can send and receive events obviously, like almost all services<br>

We can send the events to queues, to apps, to basically whatever you want.<br>
Just like in SNS there are `topics`, `subscriptions` and `event handlers` (apps or services reacting to the events).

Event Grid is for eventing and PubSub in general.<br>
I don't think it has guaranteed delivery and the order may be garbled.<br> 

### Create an Event Grid Subscription for a built-in topic
Subscription requires topic, filter (optional) and an event handler (optional)<br>
We created a storage account with a queue in it that was our event handler, i.e. the endpoint Bit of confusion with the naming there I think.<br>
We set the topic as a resource group's, to track its events.<br>
Then we set a filter to only subscribe to delete event, so only delete events would be forwarded to the storage queue.

### Create an Event Grid custom Topic (Endpoint)
Create eventgrid topic - this creates and endpoint so you obviously need to be unique globally<br>
We prepare an event handler next, i.e. we create something that will receive the notification after it has been processed by Event Grid.<br>
Lastly, we create the Event Grid subscription to tie the two parts together.

## Event hub - Microsoft's Kafka
Made for big data. Typically used to consume stuff from distributed software and devices.<br>
This is Microsoft's version of kafka.<br>
So it's like a massive ingestion queue I think and then you can fish events from partitions (queues).

|-> Main logical unit is an `Event Hub namespace`.<br>
|--> Then you create Event Hubs within that namespace.<br>
|---> Each Event Hub then has one or many partitions.

The recipients/consumer groups will read from the partitions

In relation to kafka:<br>
|Event Hub|      Kafka|
|---|---|
|namespace|      = cluster|
|event hub|      = topic|
|partition      |= partition| 
|consumer group| = consumer group|
|offset      |   = offset|

We did a cool little lab with a sender and a receiver showing the posting and receiving of events.<br>
We had an app that sent events to Event hub that split the events into four partitions<br>
We then had another app that consumed the events from the partitions<br>
Next time you had the same app in the same consumer group fetch from the partitions it of course receives nothing as it has already gotten all the events from the partitions.<br>
Other consumer groups would still get the messages once of course.

## Queue Storage (storage account feature) - a cheap happy-go-lucky version of Service Bus
A cheaper version of the `Service Bus` that does not care about the order of messages, and duplicate messages arrive. It will however guarantee at-least-once delivery<br>
So this is built into the Storage Accounts.<br>
You will have to delete messages yourself here.<br>
Once you've read a message it will be marked as invisible in the queue, but it will still be readable, it's just obscured by being invisible.<br>

Feature comparison between [Queue Storage and Service Bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted)

## Service Bus - Enterprise messaging
This is for important messages that the sender expects the consumer to process.<br>
These are for very important, high-value enterprise messaging, like financial transactions.<br>

Design patterns are found at [microsoft docs](https://docs.microsoft.com/en-us/azure/architecture/patterns/category/messaging)

Some highlighted use-cases/patterns:
#### Queues
FIFO queues with guaranteed delivery and message order intact.<br>

#### PubSub
Asynchronous delivery to subscribers in different consumer groups, very much like the `Event Hub`.<br>
But here with guaranteed delivery and event order. 

### Create a Service Bus
|-> Top level has an endpoint so it must be unique
|--> Queue (FIFO) is a sub-level resource. You must choose its size and TTL of messages
|--> Topics (PubSub) is another sub-level resource. 

For both Queue and Topics you give access to apps through `Shared Access Policies` where you as per our demo set ConnectionString for admin access.<br>
Again, this will always be more secure and more professional through Key Vault and Managed identities or app registration (well app registration sort of sucks).<br>

# Notification Hub - Push notification services
Push notification services exist to centralise push notifications from apps to end users like phones<br>
This becomes complex if you do this yourself as you on the app side have to code for Apple, Microsoft, Google, Amazon.<br>
This is where the `Notification Hub` comes into play. It does a lot of that automatically for you<br>.
Notification hub uses a standardised SDK that is all you have to work with, which simplifies push notifications a lot.

# Azure Monitor (Cloudwatch equivalence)
Azure Monitor acts much like Cloudwatch.<br>
For monitoring it has insights, visualisation and analytics<br>
For responses it can be used for (alerts/autoscale) and integrations (event hubs, logic apps, ingeset/export APIs).<br>

#### monitoring
Logs is best viewed in the log analytics. I think this may be the paid feature... but hmm not sure about that. I may have been thinking about log insights when writing that.<br>
Metrics explorer is used to view metrics obviously.<br>

#### Alerting
Activates on specified values like 60% CPU for example and can send emails and autoscale

### Activity log - Like CloudTrail
Found in the blade menu. This is available centrally within Azure Monitor. You can't delete these events.<br>
The Activity Log is available per Resource Group or per Resource as well in their blade menus.

### Application Insights
This is a paid service as you store the data in a storage account.<br>
Dashboards ad infinitum can be used like power bi, visual studio, rest apis etc.

Observes usage in near real-time for webpage or apps, gathers performance metrics.<br>
In the case of .NET you will download NuGet pacakges and then you send information together with an Instrumentation Key to application insights, to share the app data with application insights<br>
You had to do a few things on the frontend (Javascript) and some enablement on the backend too.<br>
You can now send custom messaging to the application insights in your code with simple log messages,on top of the system-defined metrics.<br>

This should be without any performance impact kind of, until you want to send full payloads to application insights.<br>
By default you will only send metadata of the performance obviously. But you can forward everything including payloads too.<br>

Azure will automatically detect which components that are talking to each other when you enable this for your apps.<br>

This is supported in like all languages you may think of.<br>

This gives you a huge amount of data of the app that you've integrated, like Failures, availability, performance etc.<br>

#### Querying the data
There are some samples for you ready to use in the Custo(?) querying language.<br>
Or you just write your own queries. You can then export the data and use like Power BI to get some nice business information.

# Transient Errors
Best practice for cloud is in general to keep retrying any requests because it's a cloud.<br>
So there will be stuff failing at any time so make your apps tolerate that.