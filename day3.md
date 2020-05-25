# Storage Accounts
(PaaS, i.e. you don't have to worry about scaling, licenses etc)<br>
Limitation of 20k IOPS and 500TB per storage account<br>
So it is a good idea to break up the storage accounts to not hit these limits.

A storage account provides the endpoint, this is a key differentiator to AWS' storage offerings.<br>
So the storage account is a top level logical unit that contains one or more categories of storage.<br>

When you create a storage account you can choose general purpose, blob only or file share.<br>
Each of them are specialised for their own usage. If you use only blobs, select the blob option. It will give you better performance the instructor said.<br>
However, note how the blob only option is only available in two regions!

It has four different categories underneath:
* File Shares - file share in the cloud. SMB and Rest. 
* Blob service - highly scalable rest-based cloud object store. also block blobs and page blobs and append blobs
* Tables (massive auto-scaling noSQL store)
* Queues reliable queues for cloud services

# Blob Storage (s3 equivalence)
The  logical unit within the storage account for blob storage is called `container`, this is sort of equivalent to an `s3 bucket`.<br>
The objects you are uploading are the `blobs`.

blobs are great for:
*serving images, storing files, streaming video and audio. All reachable via HTTP/S.

In general the same features like s3 such as versioning, encryption enforcing etc is available here too.

### Replication for blobs
[Official docs for storage redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy)
* LRS - Locally redundant with three copies in one DC
* ZRS - redundant with three copies in three DCs
* GRS - (LRS + single DC in Asynchronous region) Geo-redundant storage (GRS) copies your data synchronously three times within a single physical location in the primary region using LRS. It then copies your data asynchronously to a single physical location in a secondary region that is hundreds of miles away from the primary region
* ROGRS - Same as previous but with a secondary endpoint to read from in the asynchronous region.
* GZRS - (ZRS + single DC in Asynchronous region) Geo-zone-redundant storage (GZRS) combines the high availability provided by redundancy across availability zones with protection from regional outages provided by geo-replication. Data in a GZRS storage account is copied across three Azure availability zones in the primary region and is also replicated to a secondary geographic region for protection from regional disasters.

### Access Tier Options (Like Glacier etc)
Cool/Hot/Archive

### Options per blob
You can set options per blob upon upload. Such as the access tier, block size etc.<br>

### Types of blobs
* Block blobs (set of blocks (up to 100Mb) that together make up an object). Up to 4.75TB per object. Larger blocks so not so great for appending them. You can modify an existing block blob by inserting, replacing, or deleting existing blocks. After uploading the block or blocks that have changed, you can commit a new version of the blob by committing the new blocks with the existing blocks you want to keep using a single commit operation.
* Append blobs - for logs and appending. These are used for example for log insights. Uses small blocks so very good for appending.
* Page blobs - for very large files, up to 8TB per object. Ideal for virtual hard disks for VMs.

### Lifecycles for blobs
There are lifecycle rules that can be set for blobs<br>
There seems to be a json policy that you can write to set this up, you can of course generate this through the gui

### Events for blobs
Just like in aws' s3 buckets you can use event triggers with filters per container or file typ etc.

### Programmatic access to blob storage
You can use different languages to access blobs, obviously. For python they've divided the sdk into one module per services more or less.<br>
Why? [Because this](https://github.com/Azure/azure-sdk-for-python/issues/10646)

# Cosmos DB (PaaS)
Globally distributed noSQL db.<br>

### Consistency 
Five consistency levels that all affect the availability and latency
* (slower) Strong, Bounded-staleness, Session (default), Consistent prefix, Eventual (faster, often used by video games)

### APIs
SQL, Cassandra, MongoDB, Gremlin, TableAPI and more to come.<br>
These are set on an account level.

### Hierarchy/Structure
-> CosmosDB Account is a top level structure with an endpoint (i.e. must be globally unique). At this level you select your API.<br>
|--> Under the account you create databases (logical database units for capacity/performance)<br>
|----> Under the databases you have containers (again! :)) I suppose these are like tables <br>
|------> Under the containers you have documents/items (i.e. json files)

### Capacity and performance
The databases and containers can have dedicated capacity, shared capacity or auto-pilot. These are measured in `Request Units`.<br>
You can be quite flexible with how you assign the capacities, per database logical unit or per container.

### Creation of the Containers and formatting
Partition Key - use a json property name that has a wide range of values. There is a [partition key advisor](https://github.com/Azure-Samples/azure-cosmos-db-partition-key-advisor) .<br>
There are logical partitions that map data together based on a json key.<br>
There are physical partitions where logical partitions can point to physical storage to be grouped by, I think.<br>

Best practices if you do this manually<br>
Spread the data out<br>
All data in partitioned keys will be stored in the same physical storage space.<br>
You want to distribute the data evenly across your physical partitions.

High cardinality - increase the number of unique values, whatever this means.

### Querying with SQL style syntax
So the gui will query the documents you have selected in the menu I think.
The "FROM" parameter is just an arbitrary alias for the documents you have selected, you can set any name you want.
```SQL
SELECT * FROM <alias> <alias abbreviation> n.id = "Apollo 11"
SELECT n.Description FROM nasdaq n n.id = "Apollo 11"
```

### Additional functionality and force styling
You can add bespoke functions to help your queries. Like have an external functions with logic and refer to that in your query.<br>
You can also add a function that you can refer to when you upload documents, and force it to have a specific style/syntax/values.<br>
This was pretty complex and all written in .NET so don't know much about it, but just know that you can add functions to help searches, uploads and general usage of CosmosDB.

### Lab for CosmosDB
We did a pretty cool lab where we had a bicycle store web application running in a .NET app with a SQL backend.<br>
We migrated this using .NET to CosmosDB. We used CosmosDB as the backend instead of the SQL server, so preserving all functionality.<br>
Running the same .NET app (with modifications to use the CosmosDB backend instead) the same website was up and running just fine. Pretty cool.
We used the admin connection string (which is like god mode) during this lab.<br>
A better idea is to use [Resource Tokens](https://docs.microsoft.com/en-us/azure/cosmos-db/secure-access-to-data) which give you more granular control<br>
This probably goes for any programmatic (and possibly portal) access.

# Azure Cache for Redis
Quite simple and easy to get started apparently. Super super fast.<br>
Why on Azure? Well because setting up a redis cluster is not super easy.<br>
An in-memory database. Everything is in key-value pairs.<br>
As usual you try to put this in front of the SQL servers to speed up and scale up better.<br>
Usually, you are able to expand the amount of connections and their speed when using redis as opposed to normal SQL cache.<br>
Since the redis protocol is just single key-value pair lookup it is super fast. It usually sits close to your app to offload the SQL db.<br>

GET, SET, GETSET, EXISTS are some of the operations that you can use. There are of course more.

### Running out of memory
It's configurable what happens when you run out of memory. You can like clean out the oldest or whatever.

# CDN
Three providers are offering services in Azure, they are; Verizon, Akamai, Microsoft.<br>
They all offer slightly different capabilities and reports.<br>
You can also add some control and policies in the CDN, such as block users from particular countries to particular content.(called geo-filtering)<br>


### Hierarchy
create a profile first (in here you can choose your provider),br>
Then you create your cdn endpoint (you can use your own custom domain if you want).<br>
In your endpoint you define where your content comes from, from a storage account for example.