# Authenticating to and querying Microsoft Graph by using MSAL and .NET SDKs

Lab scenario
As a new employee at your company, you signed in to your Microsoft 365 applications for the first time and discovered that your profile information isn’t accurate. You also noticed that the name and profile picture when you sign in aren’t correct. Rather than change these values manually, you have decided that this is a good opportunity to learn the Microsoft identity platform and how you can use different libraries such as the Microsoft Authentication Library (MSAL) and the Microsoft Graph SDK to change these values in a programmatic manner.

Objectives
* Create a new application registration in Azure Active Directory (Azure AD).
* Use the MSAL.NET library to implement the interactive authentication flow.
* Obtain a token from the Microsoft identity platform by using the MSAL.NET library.
* Query Microsoft Graph by using the Microsoft Graph SDK and the device code flow.

AAD is Microsoft and Nasdaq
We get app id and tenant id.
The AAD is the front desk
Our password is our ID, I think.
The JWT token is the badge, don't share this on the internet!
The Microsoft graph is the classroom we have access to

### Exercise 1: Create an Azure AD application registration for our .NET app
We go to AAD, register an app and point the Redirect UI to http://localhost <br>
Redirect URL is where to send te app after auth is completed.<br>
The idea here is to get application id and tenant id relationship that we can use in our app for auth through AAD.<br>

Name: graphapp<br>
Supported account types: Accounts in this organizational directory only (Default Directory only - Single tenant)<br>
Redirect URI: Public client/native (mobile & desktop) http://localhost

```
# This is what we got from the gui, that we can now use in our app for auth
Application ID: c251b1e5-3b43-48ce-9fbe-c4887bf7dd0a
Tenant ID: c1bf8607-ef3c-4bc5-b8aa-223ab0eb6b5d
```

### Exercise 2: Obtain a MSAL token by using the MSAL .NET library.<br>
We use some code that connected to MSAL using the interactive authentication flow.<br>
This opens a browser window that prompts you to login with your microsoft account.<br>
It then asks you to confirm that the app will get access to specific data within your account.<br>
The application gets a MSAL token back, I think this is a JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties.

### Exercise 3: Query Microsoft Graph by using the .NET SDK and the MSAL token
Task 1: Import the Microsoft Graph SDK from NuGet (This is like importing the graph pip package)<br>
Task 2: Modify the code to use the Microsoft Graph SDK to query user profile information:
* no longer print the token
* Set scope as a provider, no clue what this is
* create a client to the Microsoft Graph service
* Add the following block of code to use the GraphServiceClient instance to asynchronously get the response of issuing an HTTP request to the relative /Me directory of the REST API, and then store the result in a new variable named myProfile of type User:
* Render the value of the User.DisplayName and User.Id members to the console:


Task 4: Test the updated application<br>
you will now get the device login prompt in stdout.<br>
Go to https://microsoft.com/devicelogin, and then enter the code value that you got from stdout.<br>
Sign in by using your Microsoft account.<br>
The program will now query Microsft Graph to get  some information it has on your account that you gave it access to. It will then print that to stdout<br>
In this exercise, you queried Microsoft Graph by using the SDK and MSAL-based authentication.

