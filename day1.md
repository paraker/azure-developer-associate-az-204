# Azure resource Manager templates in JSON (Like Cloudformation but also from cli)
Azure Resource Manager template (ARM templates) - json syntax.<br>
It is basically transformed into a REST API call PUT message with a REQUEST BODY.<br>
Usually you start from here https://github.com/Azure/azure-quickstart-templates <br>
You can also generate templates from the azure portal by going to settings -> export template <br>

You can execute the ARM templates from the az cli with `az group deployment create template-uri <github url>`. It's pretty cool actually as you ad-hoc can use templates rather than writing super long cli commands!

# Resource providers (like modules in Azure)
subscriptions -> Resource Providers<br>
There is a huge list of providers, basically one for each service within Azure.

### Provisioning VMs in Azure
#### VM basics
* Storage can be managed (EBS style) or unmanaged (storage account/blob)<br>
* Paying per second of the compute.<br>
* Paying per nic (pay only for outbound traffic) and per storage separately (used storage).<br>
* Naming up to 15 characters for Windows, up to 64 characters for linux.
* Payments for VMs are per "pay as you go" or through "reserved instances".

#### Unmanaged disks
Storage accounts is a PaaS (managed service) within Azure where you create Virtual Hard Disks (VHD)<br>
blob, table, queue and files is setup within this service.<br>
Then you attach that to your VM in some way. <br>
Need to consider throughput and capacity limits here. 20k IOPS limit.<br>
Pay only for what you use.<br>
Only available in a single AZ.
#### Managed disks
EBS style, regional resource. Just give me disks.<br>
Recommended to use this. No limits really here.<br>
No reason not to go with managed disks.<br>
Pay for the whole amount you've allocated, doesn't matter how much you use.<br>
It does not do EFS style shared to many just yet.<br>
To scale the storage you have to stop the VM temporarily.<br>
Available across multiple AZs.

#### Availability Options
* Pure VM (just a single VM) - 99.9% (if premium storage)
* Availability Set (One Data Center) - 99.95% A configuration of up to 3 fault domains and up to 20 update domains for VMs. You control the app yourself though, so not like an asg where the image is managed as well.
* Availability Zone (Multiple DCs within a Region) (Recommended where available) - 99.99% - At least 3 AZs. Small charge on the networking side here. Slight delay here.

## Containers
We can create ACR (Azure Container Registry) through `az acr create`.<br>
You can protect your ACR with IAM and also with Network Endpoints.<br>
We can run images straight on to the Azure Container Instance (PaaS) by `az container create` (this is the equivalence of ECS).

You can also have the ACR build the image for you by using a build agent.
```az acr build --registry $acrName --image myimage:latest .```

Download and push container to ACR
```bash
docker run hello-world
docker login myacrpaak.azurecr.io
# admin user is always same name as the registry (myacrpaak). Password is found in the gui under access keys for admin
# For better logins with service-principals and other things we will get back to that later 
docker tag hello-world myacrpaak.azurecr.io/hello-world
docker push myacrpaak/azurecr.io/hello-world
```
Deploy to Azure Container Instances
```bash
az container create --name nginx-demo \
--resource-group ManagedPlatform \
--image myacrpaak.azurecr.io/hello-world

Image registry username: myacrpaak
Image registry password:
 - Running ...
```
