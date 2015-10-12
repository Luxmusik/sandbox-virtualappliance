Sandbox Server 2.0 Install Guide
===========================================
Sandbox Server is the on-premises solution for enterprise teams. It allows everyone in your organisation to easilly collaborate on your stubs and mock services. This document describes how to install and configure Sandbox Server in your environment.

1. System Requirements
----------------------
#### Hardware

- If you are evaluating Sandbox, we recommend that you use a server with at least 2gb memory.
- For production, we recommend 2+ processor cores and 4gb+ memory
- For Amazon Web Services (AWS) the virtual machine needs 4gb+ memory, the equivalent of an AWS m3.medium or larger.
- The hardware requirements for a full production deployment depend on the number and frequency of stub / mock service requests and the number of active sandboxes.

#### Operating Systems

Although Sandbox can be urn on Windows, Linux and Mac systems, for enterprise use we only recommend, and support, Linus. This recommendation is based on our own testing and experience with using Sandbox.

- Sandbox is a pure Java application and should run on any platform (Apple Mac OS X, Linux, Microsoft Windows), provided all the Java requirements are satisfied.
- We recommend using a Linux environment running kernel 3.15 or above.
- In production environments Sandbox should be run from a dedicated user account.

#### Java

- Sandbox requires a very specific version of Java 8; it is JRE 1.8.0u25. We recommend using Oracle JRE 8 which you can download from the Oracle website.
- Sandbox only requires the Java JRE, not the JDK.
- 
#### Databases

Sandbox supports H2 and PostgreSQL databases.

- Sandbox bundles H2 by default and is only intended for evaluation use.
- It is recommended that Sandbox is connected to an external PostgreSQL database.

#### Ports

By default, Sandbox uses the following ports:

| Type      | Protocol  | Port Range    |
|-----------|-----------|---------------|
| Web / Combined*      | TCP       | 8080            |
| Proxy | TCP | 10000          |
| Git | TCP | 9000          |
| API | TCP | 8005          |

The 'Combined' server is a component that routes your request to the correct component (API, Git, Proxy or Webserver) based on the URL and hostname of the request URL. This means you should really only ever have to communicate with the Combined port, but the other ports will be open and thus need to be available. These ports can be changed in the Sandbox config file.

#### DNS Provider
The appliance requires a host name provided by a a DNS service. This is necesary to properly route traffic to sandboxes and system components on the appliance.

If you configure an "A" record to map the host name to an IP address you must specify a *wildcard domain* such as ```*.sandbox-va.com``` so that requests for subdomains such as ```git.sandbox-va.com```, ```bar.sandbox-va.com``` are routed to the host name.

If you configure a "C" record to map the host name to another (canonical) domain name again you must specify a *wildcard domain* such as ```*.sandbox-va.com```

The DNS hostname settings should be configured in the appliance config file, by default under the conf/ directory.

#### SMTP
The appliance uses email to send event notifications such as user invites, password resets, team changes etc and requires an SMTP server to do this. If no SMTP server is available, the appliance will continue to function however notifications will be disabled. The SMTP settings should be configured in the appliance config file, by default under the conf/ directory.

#### Setup

You will be prompted when you first execute the appliance binary to setup an Administrator account, team and enter your license. This is completed on the command line and most of the values can be updated later via the Web UI if you get something wrong. Once this step is completed you won't be prompted for the details again.

2. Upgrading the Appliance
-------------------------

New appliance versions will be shipped as new packaged zip files, just like the original installation. An upgrade can be completed by stopping the running appliance and overwriting the installation directory with the new files.

By default the persistent data like Git repositories and the application database are stored in the data/ directory inside the installation directory, for ease of upgrading later it is recommended to move this data directory to outside of the installation directory. This can be configured in the appliance config file, by default under the conf/ directory.

3. User Management
------------------
Once the appliance is configured, you can connect to the Sandbox application. A single administrator user is created for you during the configuration process; you will need to log in with the administrator's email or the username that was generated.

**Add new users:**

1. From the Dashboard, click your username in the top right-hand navbar to reveal a drop-down menu and click 'Plans & Usage'
2. Select {YourOrganisationName} from the left hand navbar
3. New users are added by inviting them to join your team. Provide a valid email address for the user and click 'Add User'.
4. An invite email will be sent to the new user with an activation link. The user is given a default password of 'password'. *Note*: If an email service provider wasn't configured no email is sent and the user should login using their email and the default password.
5. The newly added user will be listed as a member of the team

**Adding additional administrator users:**

Follow the steps above to create a user. Move your cursor over the user to reveal the available actions. Select the 'Make this user an admin' action from the options to add the 'Owner' role to the user.

**Deleting users:**

Move your cursor over the user to be deleted. If you have the appropriate administrative privileges the 'Delete' action will be available. You will be asked to confirm the delete to prevent accidental deletion.

**Password reset:**
To do password reset you must have an email service provider configured. A user can request to reset their password from the Sign-in page. An email with a reset password link will be sent to their email address.

4. Getting started with Sandbox
-------------------------------------------------

Once you have added users to the Sandbox application they are able to start building and running sandboxes on the appliance. Please refer to the Sandbox application documentation available at http://*your_sandbox_hostname*/docs for guides, examples, and API references.

5. Sandbox API
--------------------------------------------------
Sandbox exposes a RESTful API that can be used to programmatically perform user management, create and update sandboxes, and so on. The API is secured and you must provide a valid API session token to authenticate your API calls. The API expects request data encoded as JSON and returns all data as JSON.

#### Acquiring a session token
You need a session token to perform any authenticated API call such as creating new users, modifying sandboxes etc. Get a session token using the /sessions service:

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
curl -X POST -H "Content-Type: application/json" 
-d '{"usernameOrEmail":"michaelbluth", "password":"bananastand"}' http://bluth-sandbox.com/api/1/sessions
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
To create a new user you 'invite' a user to the organisation registered during the inital appliance configuration step. If you have configured an email service provider, it will send that user an activation email with a link to complete their user setup. If you do not have an email service provider configured, an email will not be sent and a default password of 'password' will be assigned to the user. The new user will be forced to change it on first login.

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
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734" 
-d '{"email":"bobloblaw@bluth.com"}' http://bluth-sandbox.com/api/1/orgs/Bluth/members
```

The service returns an empty 200 response if successful or an error if it failed.

#### Creating a sandbox
Creating a new sandbox creates a Git repository to host your code and assigns access permissions. Once created, you can pull and push code to the Git repository to update the sandbox and deploy the changes.

Create a sandbox using the /sandbox service:

```
Endpoint: /api/1/sandboxes
Method: POST
Content-Type: application/json
Cookie: sessionId={your_session_token}
Request Body:
{ 
    "ownerOrganisationName": your organisation name,
    "name": (Optional) give your sandbox a name,
    "commitBaseTemplate": (Optional) expects a boolean (true|false). If false, creates a bare repo. If true, commits example code. Defaults to true.
}
```

If you don't provide a name for the sandbox, a name will be generated for you. A curl example to create a new sandbox owned by the Bluth organisation:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734" 
-d '{"name":"banana-stand", "ownerOrganisationName":"Bluth"}' http://bluth-sandbox.com/api/1/sandboxes
```

The service returns the name of the created sandbox or an error if there was a problem processing the request:

```
{
    "name": "banana-stand",
    "ownerOrganisation": {...}
}
```

Git repositories are available on the ```git``` subdomain of the configured domain ie. ```git.bluth-sandbox.com```.  For example, the Git URL to clone the 'banana-stand' sandbox is ```http://michaelbluth@git.bluth-sandbox.com/banana-stand.git```.

#### Cloning a Sandbox
Cloning sandboxes is a great way to share web service mocks across teams. A cloned sandbox does not have its own codebase, they share the same codebase as their parent, and therefore they have no Git repository. Changes made to the parent are reflected in all clones. They run in isolation and do not share persisted data.

Clone a Sandbox using the /sandboxes service:

```
Endpoint: /api/1/orgs/sandboxes
Method: POST
Content-Type: application/json
Cookie: sessionId={your_session_token}
Request Body:
{ 
    "name": give your sandbox a name (optional),
    "ownerOrganisationName": your organisation name,
    "parentSandboxName": name of the sandbox to clone
}
```

If you don't provide a name for the cloned sandbox, a name will be generated for you. A curl example to create a clone of the ```banana-stand``` sandbox owned by the Bluth organisation:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734" 
-d '{"parentSandboxName":"banana-stand", "ownerOrganisationName":"Bluth"}' http://bluth-sandbox.com/api/1/sandboxes
```

The service returns the name of the cloned sandbox or an error if there was a problem processing the request:

```
{
    "name": "glittering-lake-4203",
    "ownerOrganisation": {...},
    "repository": {...}
}
```


6. Migrating Sandboxes Between Appliances
-----------------------------------------------------
By default, Sandbox Appliances are self contained and run in isolation. If you are running multiple appliances and you wish to migrate sandboxes from one to another then you can do this either manually or script the process. The steps are:

1. Git clone the source sandbox to a filesystem
2. Create a new target sandbox on the target appliance
3. Add a new Git remote pointing to the target sandbox
4. Push changes to the target

A fully worked example: Two environments, Development and Test, each with their own appliance. Let's call them Development appliance and Test appliance. The 'banana-stand' sandbox is on the Development appliance and we wish to programmatically migrate the sandbox to the Test appliance. The Test appliacne doesn't yet have a sandbox to host the code so we need to create one.

#### Git clone the source sandbox to a filesystem

The appliance makes Git repositories available over HTTP on the ```git.*``` subdomain, for example if the appliance is configured with Domain Name ```bluth-dev-sandbox.com``` then Git is at ```git.bluth-dev-sandbox.com```. To clone our source 'banana-stand' sandbox:

```
git clone http://michaelbluth@git.bluth-dev-sandbox.com/banana-stand.git
```

#### Create the new target sandbox

Create the target sandbox on the Test appliance with the same name via the API. We'll create it with a bare Git repository. You will need a valid API session to create the sandbox. Using our example from earlier, using Curl:

```
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734" 
-d '{"name":"banana-stand", "ownerOrganisationName":"Bluth", "commitBaseTemplate": false}' http://bluth-test-sandbox.com/api/1/sandboxes
```

This will create a new Git repository to host your code. The Git url will be ```git.bluth-test-sandbox.com/banana-stand.git```

#### Add a new Git remote pointing to the target sandbox

Having created the new target sandbox we need to add it as a git remote to the local copy of the source sandbox. For example, you can do this from a terminal with **git remote add**:

```
git remote add target-appliance http://michaelbluth@git.bluth-test-sandbox.com/banana-stand.git
```

#### Push changes to the target

Finally, push the changes to the target repository:

```
git push target-appliance master
```

And we're done. Pushing to Git will update the running sandbox with the new codebase. A sample script incorporating the steps:

```
#!/bin/bash
# clone the source sandbox code
git clone http://michaelbluth@git.bluth-dev-sandbox.com/banana-stand.git

# create target sandbox
curl -X POST -H "Content-Type: application/json" -H "Cookie: sessionId=s-db31478d-a6f8-4717-bc5a-2e587d8a7734" 
-d '{"name":"banana-stand", "ownerOrganisationName":"Bluth", "commitBaseTemplate": false}' http://bluth-test-sandbox.com/api/1/sandbox

# add new git remote
git remote add target-appliance http://michaelbluth@git.bluth-test-sandbox.com/banana-stand.git

# push changes to the target
git push target-appliance master
```

**Troubleshooting:**

*All sandbox names are globally unique for each appliance. If you try to create a sandbox with a name that is already registered you will get an error.
