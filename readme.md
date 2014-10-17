Sandbox Virtual Appliance 1.0 Install Guide
===========================================
This document describes how to install and configure the Sandbox Virtual Appliance in your environment.

1. System Requirements
----------------------

#### Amazon AWS
The appliance is made available as an AWS AMI to allow for quick and easy deployments on an Amazon EC2 instance. This means rather than having a blank Linux image, the Sandbox application will already be installed and running once Amazon creates your EC2 instance. 

The private AMI for Sandbox is available in the following regions:

* ap-southeast-2 (Asia/Sydney): ami-151a7a2f

Note that as of this time the AMI is only available in the above regions; if you wish to run your EC2 instance in another zone, please contact us for assistance.

We recommend selecting an m3.medium or larger instance. Generally speaking, the appliance benefits from a greater memory allocation than cpu.

You must attach a security group to the instance with the following inbound rules:

| Type      | Protocol  | Port Range    |
|-----------|-----------|---------------|
| SSH       | TCP       | 22            |
| HTTP      | TCP       | 80            |
| Custom TCP Rule | TCP | 1080          |

The appliance administration console is accessible on port 1080. The Sandbox application is accessible on port 80. SSH is required to apply update packages to the appliance.

Both port 22 and 1080 are used for adminstrative purposes only. For security reasons, these should not be publicly accessible. Restrict access using a firewall, or binding the ports to white-listed internal ips when creating the security group.

#### DNS Provider
The appliance requires a host name provided by a a DNS service. This is necesary to properly route traffic to sandboxes and system components on the appliance.

If you configure an "A" record to map the host name to an IP address you must specify a *wildcard domain* such as ```*.sandbox-va.com``` so that requests for subdomains such as ```git.sandbox-va.com```, ```bar.sandbox-va.com``` are routed to the host name.

If you configure a "C" record to map the host name to another (canonical) domain name again you must specify a *wildcard domain* such as ```*.sandbox-va.com```

You will need to provide the allocated host name to the appliance during the configuration process.

#### SMTP
The appliance uses email to send event notifications such as user invites, password resets, team changes etc and requires an SMTP server to do this. If no SMTP server is available, the appliance will continue to function however notifications will be disabled.


2. Configuring the Appliance
----------------------------
The appliance needs to be configured before you can commence using it. You can configure the appliance using the administration console or programmatically via the API.

#### Appliance configuration using administration console

Connect to the administration console on port 1080 to run the configuration wizard. The configuration wizard is displayed when the appliance is in its initial unconfigured state; once the configuration wizard is completed you will be redirected to the Sandbox web application.

The configuration wizard will guide you through the setup process. You will need to configure:

1. **Organisation name**: This is used for managing user access permissions. All users will be added to your organisation when they are invited to use the Sandbox.
2. **An administrator user**: This user has full administrative access over the VA. You can create further administrator users after the initial configuration is complete.
3. **DNS Hostname**: Enter the DNS hostname that you have reserved to point at the EC2 instance running the VA.
4. **Enabling emails (optional)**: Providing connection details for an email service provider is recommended so that users can receive notifications when their account is created, added to teams, projects etc and forgotten password functionality is enabled. If you do not have an internal email service provider you can setup an account on a 3rd party provider, such as Gmail, and use them as your email service provider.
5. **Enter License Key (optional)**: If you have purchased a license key you can enter it now. If you don't, the appliance operates in a free tier mode which still provides full functionality however the number of service mocks you can deploy is limited. License keys can be added later.

That's it. Save the configuration and the appliance will restart itself with the updated configuration and you will be redirected to the Sandbox web application. You are now ready to start adding users and creating mock services.

#### Appliance configuration using API

The appliance exposes an administrative RESTful API on port 1080 that can be used to programmatically configure the appliance. When the appliance is in its initial unconfigured state the API is unsecured (as the administrator user does not yet exist). Configure the appliance using the /config service:

```
Endpoint: /api/1/config
Method: PUT
Content-Type: application/json
Request Body:
{ 
    "organisationName": *Required*. The name of your organisation. All users are added to the organisation when created.
    "adminFirstName": *Required*. Administrator's first name. Note that the username will be generated from the provided first and last name.
    "adminLastName": *Required*. Administrator's last name. Note that the username will be generated from the provided first and last name.
    "adminPassword": *Required*. Administrator's password.
    "adminEmail": *Required*. Administrator's email address.
    "hostname": *Required*. The DNS hostname assigned to the appliance server. For example, getsandbox.com. Do not include http:// in the hostname.
    "emailScheme": *Required*. Value must be one of: none, smtp, smtps, SMTP, SMTPS.  All subsequent email fields must be provided to enable email. Use 'none' to turn email off.
    "emailHost": (Optional). The email service provider hostname. For example, gmail.com.
    "emailUsername": (Optional). The username to authenticate with the email service provider.
    "emailPassword": (Optional). The password to authenticate with the email service provider.
    "emailFrom": (Optional). The email address set in the 'from' field for emails.
    "license": (Optional). If you have purchased a license then provide the key here.
}
```

The /config service returns an empty response with status 200 if the configuration was successful or an error if it was not. The appliance will immediately restart itself on receipt of a valid configuration, please allow upto 120 seconds for all components to restart after the /config service has responded.

3. Updating the Appliance Configuration
---------------------------------------

The appliance configuration can be updated using the administration console or programmatically via the API.

#### Update appliance configuration using administration console

The appliance configuration can be updated by logging into the appliance administration console on port 1080. All settings except the default administrator user and the organisation can be updated. Saving the configuration will once again restart the appliance with the updated configuration and you will be redirected to the Sandbox application page.

#### Update appliance configuration using API

Once configured, the administrative RESTful API on port 1080 is secured and you must acquire a session token to perform administrative actions. The session token must be provided as a Cookie header on subsequent requests to the API.

Get a session token using the /sessions service:

```
Endpoint: /api/1/sessions
Method: POST
Content-Type: application/json
Request Body:
{ 
    "usernameOrEmail": *Required*. Administrator account username or email address,
    "password": *Required*. Administrator account password
}
```

A curl example to get a session token:

```
curl -X POST -H "Content-Type: application/json" 
-d '{"usernameOrEmail":"michaelbluth", "password":"adminPassword"}' http://bluth-sandbox.com:1080/api/1/sessions
```

The service returns a sessionId if the user credentials are correct or an error if they are not:

```
{ 
    "sessionId": "s-db31478d-a6f8-4717-bc5a-2e587d8a7734",
    "user": {...}
}
```

A request can now be sent to the /config service with the session token to update your appliance config. The following settings can be updated:

```
Endpoint: /api/1/config
Method: PUT
Cookie: sessionId=<your_session_token>
Content-Type: application/json
Request Body:
{ 
    "hostname": (Optional). The DNS hostname assigned to the appliance server. For example, getsandbox.com. Do not include http:// in the hostname.
    "emailScheme": (Optional). Value must be one of: none, smtp, smtps, SMTP, SMTPS.  All subsequent email fields must be provided to enable email. Use 'none' to turn email off.
    "emailHost": (Optional). The email service provider hostname. For example, gmail.com.
    "emailUsername": (Optional). The username to authenticate with the email service provider.
    "emailPassword": (Optional). The password to authenticate with the email service provider.
    "emailFrom": (Optional). The email address set in the 'from' field for emails.
    "license": (Optional). If you have purchased a license then provide the key here.
}
```

The /config service returns an empty response with status 200 if the configuration was successful or an error if it was not. The appliance will immediately restart itself on receipt of a valid configuration, please allow upto 120 seconds for all components to restart after the /config service has responded.

4. Upgrading the Appliance
-------------------------
The appliance can be upgraded with service packs. Service packs are made available periodically to address security fixes and provide feature updates.

Latest service pack: [Service Pack v1.0.1](https://s3-us-west-2.amazonaws.com/getsandbox-assets/1413495704-appliance.sbx)

**Applying service packs:**

To apply a service pack:

1. Take a snapshot of your appliance as a backup.
2. Download the service pack to your local filesyastem
3. Copy the service pack to the appliance via scp. It must be copied to the sandboxadmin users home directory. For example:```
scp scp://sandboxadmin:sandbox@<your-appliance-ip>:22/~/ ./1413495704-appliance.sbx```

4. Log into the Adminstration console on port 1080 and scroll down. If the service pack was successfully copied a 'Local update found' message will be displayed.
5. Clicking 'Install now' will apply the service pack and restart the appliance. Restart can take up to 2 minutes.

**Troubleshooting**

1. If you have uploaded an invalid service pack you will receive an error alert on the adminstrative console and the upgrade process will be aborted. The appliance will remain in its previous state.

2. If on restart, the appliance is in an inconsistent state, restore from your backup snapshot and contact [support](mailto:support@getsandbox.com) for assistance.

4. User Management
------------------
Once the appliance is configured, you can connect to the Sandbox application. A single administrator user is created for you during the configuration process; you will need to log in with the administrator's email or the username that was generated.

**Add new users:**

1. From the Dashboard, click your username in the top right-hand navbar to reveal a drop-down menu and click 'Organisations'
2. Select {YourOrganisationName} -> Members from the left hand navbar
3. New users are added by inviting them to join your organisation. Provide a valid email address for the user and click 'Add User'.
4. An invite email will be sent to the new user with an activation link. The user is given a default password of 'password'. *Note*: If an email service provider wasn't configured no email is sent.
5. The newly added user will be listed as a member of the organisation

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
