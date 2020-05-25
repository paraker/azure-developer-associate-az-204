## What the course is about
Cloud developer role - design build, test and maintain cloud solutions such as apps and services.<br>

## Prerequisites
* Expecting at least one year of developing scalable solutions through software development.
* Focuses on C#, .NET, HTML using REST
* Foundational understanding of Azure cloud
* Azure cli and or powershell

## Course modules
[Day 1](day1.md)
* ARM templates
* Resource providers in Azure (like application modules for the cloud)
* implement IaaS solutions (VMs/Containers)

[Day 2](day2.md)
* Creating azure app services web apps (PaaS)
* implement azure functions
* implement API management (API Gateway)

[Day 3](day3.md)
* develop solutions that use blob storage
* develop solutions that use cosmos db storage (Cassandra/DynamoDB)
* Integrate caching and content delivery (Redis + Microsoft)

[Day 4](day4.md)
* implement user authentication and authorization
* implement secure cloud solutions
* Develop App Service Logic Apps

[Day 5](day5.md)
* Develop event-based solutions (Like SNS)
* Develop message-based solutions
* Instrument solutions to support monitoring and logging (CloudWatch)

Labs still to do; 10, 11, 12
There are challenge labs to do at tsfb.learningondemand.net link

Tips for the exam:
* The Container Instances are created with az cli through "az container create"
* When making queries for CosmosDB you need aliases to be able to refer to it in your filters.
* You can create policies and attach Service Shared Access Signatures.
* Logic Apps connectors for `while loops` is `Until`. `Condition` is for `if/else`, `foreach` is for `foreach` I suppose.

