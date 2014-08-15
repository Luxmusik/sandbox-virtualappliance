Sandbox Virtual Appliance Installation Manual
=============================================

This document describes how to install and configure the Sandbox Virtual Appliance (VA) in your environment.


1. System Requirements
----------------------
The VA is made available as an AWS AMI to allow for quick and easy deployments of Sandbox on an Amazon EC2 instance. This means rather than having a blank Linux image, Sandbox will already be installed and running once Amazon creates your EC2 instance. 

The private AMI for Sandbox is available in the following regions:

* ap-southeast-2 (Asia/Sydney): ami-2bf89e11

Note that as this time the AMI is only available in the above regions; if you wish to run your EC2 instance in another zone, please contact us for assistance.

We recommend selecting an m3.medium or larger instance. Generally speaking, the VA benefits from a greater memory allocation than cpu.

Once launched, you will need to modify the security group applied to the instance to allow incoming connections on ports 80 and 1080. After updating the security group, you will be able to connect to the administration console on port 1080.

*DNS Provider??*
*SMTP??*

2. Configuring the VA
---------------------
The VA needs to be configured before you can commence using it. Connect to the administration console on port 1080 to run the configuration wizard. The configuration wizard is displayed on the first run; once configured you will be taken to the standard administration console.

The configuration wizard will guide you through the setup process. You will need to configure:

1. *Organisation name*: This is used for managing user access permissions. All users will be added to your organisation when they are invited to use the Sandbox.
2. *An administrator user*: This user has full administrative access over the VA. You can create further administrator users after the initial configuration is complete.
3. *DNS Hostname*: Enter the DNS hostname that you have reserved to point at the EC2 instance running the VA.
4. *Enabling emails (optional)*: Providing connection details for an SMTP server is recommended so that users can receive notifications when they are added to teams, projects etc and forgotten password functionality is enabled. If you do not have an internal SMTP server you can setup an account on a 3rd party provider, such as Gmail, and use them as your SMTP server.
5. *Enter License Key (optional)*: If you have purchased a license key from us you can enter it now. If you don't, the VA operates in a free tier mode which still provides full functionality however the number of service mocks you can deploy is limited.

That's it. Save the configuration and the VA will restart itself with the updated configuration. The next time you visit the administration console on port 1080 you will be presented with a login screen. Once logged in, you can change the VA configuration, and restart system services if need be.

3. User management
------------------
Once the VA is configured, you can connect to the Sandbox dashboard on port 80 to add users. A singe administrative user is created for you during the configuration process; you will need to log in with the administrator's email or the username that was generated.

To add new users:

1. Goto 'Organisations' page accessible from your profiles drop-down menu (top right hand navbar)
2. Select 'YourOrganisationName' -> Members from the left hand navbar
3. Add a new user by inviting them to join your organisation. Provide a valid email address for the user and select Add User.
4. An invite email will be sent to the new user with an activation link. If an SMTP server wasn't configured no email is sent, instead a default password of 'password' is set and the user must log in with their email address initially.
5. The newly added user will be listed as a member of the organisation

Adding additional administrator users:

1. Follow the steps above to create a user
2. Move your cursor over the user to reveal the available actions. Select the 'Make this user an admin' action from the options to add the 'Owner' role to the user.

Deleting users:

1. Move your cursor over the user to be deleted. If you have the appropriate administrative privileges the 'Delete' action will be available. You will be asked to confirm the delete to prevent accidental deletion.

4. Creating new mock applications
-----------------------------
All users can create new mock applications and virtual clones of mock applications.

5. Configure Sandbox programmatically with the API
--------------------------------------------------
Sandbox exposes a REST API that can be used to programmatically configure mock applications, users etc. The API is secured and you must provide a valid API session token to authenticate your API calls. The API expects request data encoded as JSON and returns all data as JSON.

#### Acquiring a session token
You need a session token to perform any authenticated API call such as creating new users, modifying mock applications etc. Get a session token using the /sessions service:

```
Endpoint: /api/1/sessions
Method: POST
Content-Type: application/json
Request Body:
{ 
    "usernameOrEmail": your account username or email address,
    "password": your account password
}
```

A curl example to get a session token:

```
curl -X POST -H "Content-Type: application/json" -d '{"usernameOrEmail":"michaelbluth", "password":"bananastand"}' http://bluth.com/api/1/sessions
```

The service returns a sessionId if the user credentials were correct or an error if they are not:

```
{ 
    "sessionId": "s-db31478d-a6f8-4717-bc5a-2e587d8a7734",
    "user": {
        "firstName": "Michael",
        "lastName": "Bluth",
        "username": "michaelbluth",
        "email": "michael@bluth.com",
    }
}
```

#### Creating a user
To create a new user you 'invite' a user to the organisation registered during the inital VA configuration step. If you have SMTP configured, it will send that user an activation email with a link to complete their user setup. If you do not have SMTP configured, an email will not be sent and a default password of 'password' will be assigned to the user. It is recommended they change it on first login.

Create a user using the /orgs/{org}/members service:

```
Endpoint: /api/1/orgs/{your_organisation_name}/members
Method: POST
Content-Type: application/json
Cookie: sessionId=<your_session_token>
Request Body:
{ 
    "email": the email address of the user to invite
}
```

A curl example to create a new user for the Bluth organisation (note that organisation names are case sensitive):

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734; username=michaelbluth" -d '{"email":"bobloblaw@bluth.com"}' http://bluth.com/api/1/orgs/Bluth/members
```

The service returns an empty 200 response if successful or an error if it failed.

#### Creating a mock application
Creating a new mock application creates the Git repository to host the mock application code and assigns access permissions. Once created, you can pull / push code to the Git repository to update your mock application and deploy the changes.

Create a mock application using the /apps service:

```
Endpoint: /api/1/apps
Method: POST
Content-Type: application/json
Cookie: sessionId={your_session_token}, username={your_username}
Request Body:
{ 
    "name": give your mock application a name (optional),
    "ownerOrganisation": {
        "name": your organisation name
    }
}
```

If you don't provide a name for the mock application, Sandbox will generate one for you. A curl example to create a new mock application owned by the Bluth organisation:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734; username=michaelbluth" -d '{"name":"banana-stand", "ownerOrganisation":{"name": "Bluth"}}' http://bluth.com/api/1/apps
```

The service returns the name of the created mock application or an error if there was a problem processing the request:

```
{
    "name": "banana-stand",
    "ownerOrganisation": {...}
}
```

Git repositories are available on the ```git``` subdomain of the configured domain ie. ```git.sandbox.bluth```.  The Git URL to clone the 'banana-stand' mock application is ```http://michaelbluth@git.sandbox.bluth/banana-stand.git```.

Creating a mock application also creates an instance of your application that is deployed and running with the same name. For example, to retrieve details for the ```banana-stand``` instance created in the above example call the /api/1/instances/banana-stand endpoint.

#### Creating an instance of a mock application
You can create multiple running instances of a mock application for use by different teams. Although instances share the same codebase, they run in isolation and do not share persisted data.

Create a mock application using the /instances service:

```
Endpoint: /api/1/orgs/instances
Method: POST
Content-Type: application/json
Cookie: sessionId={your_session_token}, username={your_username}
Request Body:
{ 
    "name": give your mock application a name (optional),
    "repositoryName": the mock application to create instance of,
    "ownerOrganisationName": your organisation name
}
```

If you don't provide a name for the instance, Sandbox will generate one for you. A curl example to create a new instance of the ```banana-stand``` mock application owned by the Bluth organisation:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734; username=michaelbluth" -d '{"repositoryName":"banana-stand", "ownerOrganisationName":"Bluth"}' http://bluth.com/api/1/instances
```

The service returns the name of the created mock application or an error if there was a problem processing the request:

```
{
    "name": "glittering-lake-4203",
    "ownerOrganisation": {...},
    "repository": {...}
}
```


6. Migrating mock applications between Sandbox VAs
---------------------------------------------------
By default, Sandbox Virtual Appliances are self contained and run in isolation. If you are running multiple Sandbox VAs and you wish to migrate mock applications from one to another then you can do this either manually or script the process. The steps are:

Migrating code to a *new* mock application:

1. Git clone the source mock application to a filesystem
2. Create a new target mock application on the target VA
3. Add a new Git remote pointing to the target mock application
4. Merge the repositories and push changes to the target

Migrating code to an *existing* mock application

1. Git clone the source mock application from the source VA to a filesystem
2. Add a new Git remote pointing to the target mock application
3. Merge the repositories and push changes to the target

A fully worked example: Two environments, Development and Test, each with their own Sandbox VA. Let's call them Development VA and Test VA. There is a mock application called 'banana-stand' on the Development VA and we wish to programmatically migrate the application to the Test VA. The Test VA doesn't yet have a mock application to host the code so we need to create one.

#### Git clone the source mock application to a filesystem

The VA makes Git repositories available over HTTP on the ```git.*``` subdomain, for example if the VA is configured with Domain Name ```dev.bluth.com``` then Git is at ```git.dev.bluth.com```. To clone our source 'banana-stand' mock application:

```
git clone http://michaelbluth@git.dev.bluth.com/banana-stand.git
```

#### Create a new target mock application

Let's create the target mock application on the Test VA with the same name via the API. You will need a valid API session to create the mock application. Using our example from earlier, using Curl:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734; username=michaelbluth" -d '{"name":"banana-stand", "ownerOrganisation":{"name": "Bluth"}}' http://test.bluth.com/api/1/apps
```

This will create a new Git repository to host your mock application code. The Git url will be ```git.test.bluth.com/banana-stand.git```

#### Add a new Git remote pointing to the target mock application

Having created the new target mock application we need to add it as a git remote to the local copy of the source mock application. From a terminal we do this with **git remote add**:

```
git remote add target http://michaelbluth@git.test.bluth.com/banana-stand.git
```

#### Merge the repositories and push changes to the target

We simply want to pull in the target Git repository and over-write the default template it contains with the source repository codebase:

```
git pull --no-edit -Xours target master
```

Finally, push the changes to the target repository:
```
git push target master
```

And we're done. A sample script incorporating the steps:

```
#!/bin/bash
# clone the source mock application
git clone http://michaelbluth@git.dev.bluth.com/banana-stand.git

# create target mock application
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734; username=michaelbluth" -d '{"name":"banana-stand", "ownerOrganisation":{"name": "Bluth"}}' http://test.bluth.com/api/1/apps

# add new git remote
git remote add target http://michaelbluth@git.test.bluth.com/banana-stand.git

# merge and push changes
git pull --no-edit -Xours target master
git push target master
```


