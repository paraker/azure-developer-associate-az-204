# User and App Authentication and Authorisation 

Azure Active Directory is in the middle of almost all services' identification.

AAD manages two types of objects:
* Application registrations
* security principals - a security principal can be a `user principal` or a `service principal` (managed identities use service principles in the background)

### Application registration
"Admin access level passwords" kind of, pretty shit. Don't use this. Use managed identities.

### AAD Tenant (Top level special AAD)
This has Management groups under it
A `Directory` in the portal is equivalent to a `Tenant`.<br>
This is visible under Azure Active Directory in the portal.
https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview

This has support contracts, users, licenses etc tied to it.

### Management Groups
Used to logically group subscriptions<br>
This can be used for set roles<b, which is pretty sweet<br>
Also can be used to avoid service limits<br>

### App registrations
If the application that you are working on requires aaa you will use this.
Create one through `App Registrations`.<br>
You can tie this to a single tenant or to many tenants<br>
You will get a set of credentials and identifiers, `application (client) id`, `directory (tenant) id`, and an `object id`.

### Role assignment
You can assign roles to a user in the tentant directory on any level, i.e. Directory (tenant), Management Groups, Subscription, or RG, or resource(?).<br>
If that user is given the right role, zhe should be able to create new roles within that scope it was created for.<br>
I think that will take care of our needs on a sub level, we want to I suppose give some level of access to the users?<br>
Perhaps not, perhaps we want to just give terraform that access, like we have in AWS.

### Oauth2
You can use single or double oauth endpoints.<br>
But in general the client app talks to the Azure AD, gets a microsoft token.<br>
Then the microsoft token is used to speak to services within Azure.<br>

You can also utilise external authentication, such as using social auth like facebook or github.<br>
So when you have been authenticated to prove who you are you can speak to Azure to ask for what you have access to.<br>

### Authentication flows
* An application get
* On-behalf-of authentication flow (A service uses an application's credentials to get access to a resource)
* Client Credentials authentication flow (An admin configures AAD with creds and puts them into the app). The app can then 
* Device code authentication flow (docker-cli-tools way of authenticating)

### Active Directory Authentication Library (ADAL)
This is a library to programatically get access. But this is legacy, that we should be using instead, called MSAL.

### Microsoft Authentication Library (MSAL)
A newer library to be used by code to get authentication tokens.
Tenant, client-id and authority is sent to Azure<br>
You will get a token as a response.
This is like when apps want access to your facebook account, when they want to use your information such as photos, profile, address etc.<br>

### Microsoft Graph
Holds information about your behaviours in your Microsoft account. Crazy amount of your behaviours and uses.<br>
This can be used by Microsoft apps (and perhaps our own apps?) to integrate with

### Lab with oauth mechanisms
See [lab 6](lab6.md)

# Storage Accounts permissions
Connection Strings and Access keys give full permissions to ALL things in the Storage Account.<br>
This is not great. A better approach is Shared Access Signatures. Just like signed URLs for S3 I suppose.

### Shared Access Signatures
A better approach is Shared Access Signatures (SAS), however, watch out for [ad-hoc SAS](###Ad-hoc SAS sucks).<br>
SAS work much like signed URLs for S3 I suppose.<br>

Components of the signed URL is:
* Container URI (your container URI)
* Storage services version (2012-02-12)
* Start time (date)
* Expiration time (date)
* Resource (blob)
* Permissions (read/write)
* signature (long hash key)

These can be generated in the console (inside a storage account) or programmatically.<br>
Generally the application goes to the SAS provider to get the URL.<br>
Then the app will go to the resource directly with the SAS token it now possesses.

### Ad-hoc SAS sucks
They are just a time-based URL that anyone that has access to will be able to use.<br>

#### SAS Policy that generates tokens
This is an added security measure sort of like an intermediate certificate authority.<br>
Like if you have a breach, you can invalidate the policy to automatically invalidate associated SAS tokens.<br>

Policies can't be generated from the portal gui.<br>
However, they can be generated through the storage explorer.

### Authorisation
* Every request must be authorised
* REST API requests can use a Shared Key Authorization scheme

# Azure Key Vault (Like aws KMS)
Cloud HSM with FIPS 140-2 Level 2<br>
Note how you can create multiple key vaults. This could be useful to segregate access to keys.

### Authentication
Can be done with:
* Managed identities (like an aws iam service/instance role, good practice)
* Service principal (App registration ID that gives you a "password token" that you'll configure in your app, bad practice) 

```
# Create key with cli
az keyvault create --name --location 
az keyvault secret set --name --resource-group --location
az keyvault secret show
```
Or programatically
```
# Set your service principal auth
export AZURE_CLIENT_ID=<your-clientID>
export AZURE_CLIENT_SECRET=<your-clientSecret>
export AZURE_TENANT_ID=<your-tenantId>
export KEY_VAULT_NAME=<your-key-vault-name>
```
Python
```
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = "mySecret"

print("Input the value of your secret > ")
secretValue = raw_input()

print("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...")

client.set_secret(secretName, secretValue)

print(" done.")

print("Forgetting your secret.")
secretValue = ""
print("Your secret is '" + secretValue + "'.")

print("Retrieving your secret from " + keyVaultName + ".")

retrieved_secret = client.get_secret(secretName)

print("Your secret is '" + retrieved_secret.value + "'.")
print("Deleting your secret from " + keyVaultName + " ...")

client.delete_secret(secretName)

print(" done.")
```

### Replace a password with another password to auth to a DB
* Register an app (bad practice, use managed identities instead)
* Create a secret in keyvault (this is like the parameter store kind of) this has an identifier string that can now be used in apps
* Create an access policy for the app in keyvault. The policy will say that you can only GET secrets. This will refer to the app id that we've registered in step 1.
* Use the retrieved secret in your app

# Managed Identities (The good way, like iam service/instance roles)
`Managed identities` is using `services principles` in the background, but you don't have to worry about the passwords. 

Two options:
* System-assigned managed identity<br>
Created per Azure resource, so it is tied to a logic app, data factory, container instance etc.<br>
These cannot be shared. Can only be assigned to a single azure resource.
* User-assigned managed identity, like an iam role. Can be shared between multiple resources.<br>
These are decoupled from the resources.

### Operations for system-assigned identity
* Go to `identity` on your resource and stay on the system-assigned tab, toggle the identity (an object ID and the name of the resource itself, like the arn)
* Go to the resource that you want the first resource to have access to.
* click IAM -> add role assignment -> find your resource identity in the menus by filtering on the type of resource.

In keyvault you can now also permit access policies based on the identity we just created too, great! no more passwords!

### Operations for user-assigned identity
* Create your user-assigned resource through gui (managed identities) or cli.
* Go to `identity` on your resource and select the user-assigned tab and add the newly created user-assigned identity to this resource
* Now go to whatever resource that you want this user-assigned identity should have access to. Of course you can set your policies as you want, and the principal in the policy should be your identity.

### [Lab 7](lab7.md)
Uses a function that will use managed identities to auth to key vault to read a connection string for a storage account.

# Logic Apps (just like step functions)
GUI based logic (configured by json files).

These allegedly time out after 4 minutes, fairly surprising.<br>
The Logic Apps have a TONNE of API integrations with third parties, like bing, google, aws, facebook etc.

### Custom connectors
We can import APIs that are not yet integrated into the Logic Apps, which is pretty cool.<br>
Looked like you can do a swagger api explorer here too for your API.<br>
You will define which actions that you should be able to use in the Logic App workflow.<br>
You could also choose which type of authentication for that particular connector.

You can actually certify your custom connector with Microsoft and start selling that custom connector to others on Azure.