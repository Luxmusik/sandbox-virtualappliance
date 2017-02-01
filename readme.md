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

Although Sandbox can be run on Windows, Linux and Mac systems, for enterprise use we only recommend, and support, Linux. This recommendation is based on our own testing and experience with using Sandbox.

- Sandbox is a pure Java application and should run on any platform (Apple Mac OS X, Linux, Microsoft Windows), provided all the Java requirements are satisfied.
- We recommend using a Linux environment running kernel 3.15 or above.
- In production environments Sandbox should be run from a dedicated user account.


#### Java

- Sandbox requires a very specific version of Java 8; it is JRE 1.8.0u25. We recommend using Oracle JRE 8 which you can download from the Oracle website.
- Sandbox only requires the Java JRE, not the JDK.

To check your version of Java, in a terminal run this:

```
java -version
```

The version of Java should be "1.8.0_112". If you don't then please install the correct version.

#### Databases

Sandbox supports H2 and PostgreSQL databases.

- Sandbox bundles H2 by default and is only intended for evaluation use.
- It is recommended that Sandbox is connected to an external PostgreSQL database. Connection details are in the ```server.properties```.
- We support PostgreSQL 9.1+


### Web Browsers

- Chrome: latest stable version supported
- Firefox: latest stable version supported
- Internet Explorer: Support for version 11+. Previous versions not supported
- Safari: latest stable version supported

#### Ports

By default, Sandbox uses the following ports:

| Type      | Protocol  | Port Range    |
|-----------|-----------|---------------|
| Web / Combined*      | TCP       | 8080            |
| Proxy | TCP | 10000          |
| Git | TCP | 9000          |
| API | TCP | 8005          |

The 'Combined' server is a component that routes your request to the correct component (API, Git, Proxy or Webserver) based on the URL and hostname of the request URL. This means you should really only ever have to communicate with the Combined port, but the other ports will be open and thus need to be available. These ports can be changed in the Sandbox properties file (see Installation section for details).

#### DNS Provider
Sandbox Server requires a host name provided by a a DNS service. This is necesary to properly route traffic to sandboxes and system components on the server.

If you configure an "A" record to map the host name to an IP address you must specify a *wildcard domain* such as ```*.sandbox-server.com``` so that requests for subdomains such as ```git.sandbox-server.com```, ```bar.sandbox-server.com``` are routed to the host name.

If you configure a "C" record to map the host name to another (canonical) domain name again you must specify a *wildcard domain* such as ```*.sandbox-server.com```

The DNS hostname must be configured in the server properties file (see Installation section for details).

#### SMTP
The appliance uses email to send event notifications such as user invites, password resets, team changes etc and requires an SMTP server to do this. If no SMTP server is available, the appliance will continue to function however notifications will be disabled. The SMTP settings should be configured in the server properties file (see Installation section for details).

2. Installation
-------------------------

Extract the downloaded zip file to an install location. The path to the extracted directory is referred to as the ```<Sandbox installation directory>``` in these instructions. Note that you should use the same user account to both extract Sandbox and to run Sandbox to avoid possible permission issues at startup.

#### Tell Sandbox where to store your data

The Sandbox home directory is where all of you Sandbox data is stored.

The home directory defaults to ```<Sandbox installation directory>/data```. To change this, create a new Sandbox home directory (without spaces in the name), and then tell Sandbox where you created it by editing the ```<Sandbox installation directory>/conf/server.properties``` file - add the absolute path to your home directory to the ```persist.path``` attribute. Here's an example of what that could like when you're done:

```
# Sandbox home directory: all data is stored under this path.
# Set this variable to a valid path
persist.path="/Users/bob/sandbox-home"
#
```

#### Set the root domain to access Sandbox

Sandbox requires a hostname and this is configured in the server properties file. Edit ```<Sandbox installation directory>/conf/server.properties``` file - add the host name to the ```app.hostname``` attribute. Here's an example of what that could like when you're done: 


```
# Host name: set this to the root hostname designated for Sandbox. This is required to route
# requests to sandboxes as each one exists as a subdomain. For example, a sandbox called 'test'
# and a hostname of sandbox-server.com would be accessible at test.sandbox-server.com.
#
app.hostname=sandbox-server.com
#
```

#### (Optional) Configure database connection properties

Sandbox supports an H2 embedded database by default, but can be configured to use a more robust external Postgres database instead. If using Postgres, three separate schemas will are required. The configuration properties could look like:

```
jdbc.api.type=postgres
jdbc.api.url=jdbc:postgresql://dbserver:5432/sandbox
jdbc.api.user=username
jdbc.api.password=password
```

Once the connection properties are setup correctly, upon application start the schemas will automatically be populated and used. No other interaction is required.

#### Start Sandbox!

In a terminal, change directory to ```<Sandbox installation directory>``` and run this:

```
./bin/sandbox-server
```

You will be prompted on the command line to enter your name, choose a password, create a team name and optionally provide a license. Your administrator account is created from your name, it will be displayed on the terminal. Here's an example of what this looks like:

```
Enter administrator first name: ando
Enter administrator last name: stewart
Enter administrator email: a@c.com
Enter administrator password (input is masked): 
Enter team name (alphanumeric only, no spaces): ando
License key (will have been emailed to you, optional): 
2015-10-13 12:38:36:975 INFO  sandbox.runner.Runner -                                      - Create administrator with username: 'andostewart'
2015-10-13 12:38:37:062 INFO  sandbox.runner.Runner -                                      - Initial setup completed successfully.
2015-10-13 12:38:37:062 INFO  sandbox.runner.Runner -                                      - API service started.
2015-10-13 12:38:37:815 INFO  sandbox.runner.Runner -                                      - Git service started.
2015-10-13 12:38:37:838 INFO  sandbox.runner.Runner -                                      - Drone service started.
2015-10-13 12:38:38:639 INFO  sandbox.runner.Runner -                                      - Proxy service started.
2015-10-13 12:38:38:718 INFO  sandbox.web.Router    -                                      - Starting router for host 'lvh.me' on port 8080 with subdomains 'git.lvh.me' and 'www.lvh.me'
2015-10-13 12:38:38:723 INFO  sandbox.runner.Runner -                                      - Web server started.
2015-10-13 12:38:38:725 INFO  sandbox.runner.Runner -                                      - All components started.
```

Then, in your browser, go to ```http://<Sandbox hostname>:8080``` and sign in with your new credentials. You're ready to go, starting adding users and creating sandboxes.

3. User Management
------------------

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

5. Upgrading Sandbox
-------------------------

New Sandbox versions will be shipped as new packaged zip files, just like the original installation. An upgrade can be completed by stopping the running appliance and overwriting the installation directory with the new files.

By default the persistent data like Git repositories and the application database are stored in the data/ directory inside the installation directory, for ease of upgrading later it is recommended to move this data directory to outside of the installation directory. This can be configured in the appliance config file, by default under the conf/ directory.

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

**Troubleshooting:**

*All sandbox names are globally unique for each appliance. If you try to create a sandbox with a name that is already registered you will get an error.
