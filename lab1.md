API frontend with a storage backend that stores images that the app shows.<br>
Well, technically I changed it to a python app but anyhow.
```bash
# Explcitly create resource group
az group create --name ManagedPlatform --location centralus

# Create storage account to store images
az storage account create --resource-group ManagedPlatform --name imgstorpaak --location centralus --sku Standard_LRS --kind StorageV2 --access-tier Hot

# Get connection string of our storage account, this is used when creating the storage container to connect to the storage account
# We can use the connection string, the storage account key or a sas token to authenticate to the storage account
# Note how the connection string (and probably account key) give admin access.
# Note how there are two connection strings and two account keys, this is for rotation purposes.
az storage account show-connection-string --name imgstorpaak
"DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=imgstorpaak;AccountKey=sP308qvdpto922P8hoVzefMrlq3oEnJgcnv+I0P7QfhegedV1050FIbs87KHh9au1E8UoQxE1fPMuGWqQ3TV0w=="

# Create a storage container. Note that this is not a compute container, just a storage container.
# We are creating a storage blob in this case (equal to s3)
az storage container create --name images --resource-group ManagedPlatform --public-access blob --account-name imgstorpaak --connection-string "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=imgstorpaak;AccountKey=sP308qvdpto922P8hoVzefMrlq3oEnJgcnv+I0P7QfhegedV1050FIbs87KHh9au1E8UoQxE1fPMuGWqQ3TV0w==" --account-key "sP308qvdpto922P8hoVzefMrlq3oEnJgcnv+I0P7QfhegedV1050FIbs87KHh9au1E8UoQxE1fPMuGWqQ3TV0w=="

# Create an app service plan (asg VM)
az appservice plan create --name ManagedPlan --resource-group ManagedPlatform --sku S1

# Deploy web app on to asp, I changed to a python app here. This will obviously be deployed into the centralus location as that is where our asp is.
az webapp create --name imgapipaak --resource-group ManagedPlatform --plan ManagedPlan --runtime "python|3.6" --deployment-source-url https://github.com/Azure-Samples/python-docs-hello-world --deployment-source-branch master
```