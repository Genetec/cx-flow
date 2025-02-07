* [Namespace WebHooks](#webhook)
* [GitHub](#github)
* [GitHub App](#githubapp)
* [GitLab](#gitlab)
* [Azure Devops](#azure)
* [Bitbucket Server](#bitbucketserver)
* [Bitbucket Cloud](#Bitbucketcloud)


The url/endpoint for all webhook registrations are as follows:
<br>http://cxflow | https://cxflow 
<br>http://cxflow/cx | https://cxflow/cx 
<br>/cx is an applicable context that can be used, but the default / root context will
<br>**Note** replace cxflow with end point/port that you are running the webservice (i.e. localhost:8080)

## <a name="webhook">Namespace WebHooks</a>
WebHooks can be registered at the namespace level (Organization in GitHub, Group within GitLab, Team in Bitbucket).  This will apply the WebHook configuration globally for all Repositories underneath within the hierarchy.

## <a name="github">GitHub</a>
<br>When registering the webhook in GitHub, ensure the application/json Content type is selected.  Form URL Encoded is not supported. 
<br>The secret must be the pre-shared token that the CxFlow webservice is using to validate and authenticate requests.
<br>The supported events are Pull Request, which will by default produce feedback within the pull request itself, and Push Event, which will execute the desired bug tracker implementation. 
<br>Starting with version 1.6.0, Branch or tag deletion events are supported and will delete corresponding projects in CxSAST when scanned branches are deleted (which is frequently done after a pull request is successfully merged).
[[/Images/webhookGithub.png|Example of GitHub webhook registration]]

## <a name="githubapp">GitHub App</a>
<br>CxFlow can be created as a GitHub App.  Follow the GitHub instructions for creating an App [here](https://docs.github.com/en/free-pro-team@latest/developers/apps/creating-a-github-app)
* Make sure the GitHub App points to your CxFlow http endpoint
* Available options for Events are same as WebHook registration mentioned above (Push, Pull, Delete Branch)
* Ensure to download/save the Private key generated for the App
* You must convert the key to PKCS8 formatted PEM using: ```openssl pkcs8 -topk8 -inform PEM -outform PEM -in private.pem -out private8.pem -nocrypt```
* To make use of the GitHub App, you must add app-id and app-key-file to your configuration
* The base headers are ```application/vnd.github.machine-man-preview+json, application/vnd.github.v3+json``` to update them use the app-header config under the github block
```
github:
  app-id: XXXXX #This ID will be found in your GitHub App configuration settings
  app-key-file: /path/to/keyfile.pem
  app-headers: application/vnd.github.v3+json #Optional
  webhook-token: XXXX #Preconfigured WebHook Secret as defined in the GitHub App.
  url: https://github.com
  api-url: https://api.github.com/repos/
  #app-url: https://api.github.com/app/ This is default, only add configuration if using on-prem or non-cloud url
  block-merge: true
  error-merge: true
  cx-summary: true
```

### Permissions Required for GitHub App
The following permissions are required
  * Contents - Read/Write | For access to repository contents
  * Pull Request - Read/Write | For commenting on PR  
  * Commit statuses - Read/Write | For Block/Break Merge
  * Webhooks - Read/Write | For ability to Register WebHook Events
  * Issues - Read/Write | If using GitHub as bug-tracker  

WebHook Events
  * Delete
  * Push
  * Pull Request

## <a name="gitlab">GitLab</a>
[[/Images/webhookGitLab.png|Example of GitLab webhook registration]]

## <a name="azure">Azure Devops</a>
<br>Azure DevOps requires a different endpoint for Pull and Push events due to the fact the payload and headers cannot be differentiated. 
  * When registering Pull Create Events, use http://<cxflow>/ado/pull 
  * When registering Push Events, use http://<cxflow>/ado/push 
**Note** Only Push/Pull Create events are currently supported. Token should be sent as Basic Authentication Header.

## <a name="azureWeb">Changing Checkmarx scan submit comment status in ADO  </a>
* When registering Pull Create Events, use http://<cxflow>/ado/pull?commentmsgid=2
* commentmsgid=1 is for comment status as Active
* commentmsgid=2 is for comment status as Resolved
* commentmsgid=3 is for comment status as Won't Fix
* User can pass comment status id as command line parameter as --azure.comment-status=2
* Inside YML configuration user can put information as below
```
azure:
  webhook-token: cxflow:12345
  open-status: "To Do"
  comment-status: 2
```


## <a name="bitbucketcloud">Bitbucket Cloud</a>
Bitbucket cloud does not support a shared key/secret for digitally signing and verifying the request, so we require the token parameter to be passed:
<br> example: http://cxflow?token=XXXXX
<br>XXXX is the pre-shared token value set for `bitbucket.webhook-token` that the CxFlow webservice is using to validate and authenticate requests.

When configuring the API token in CxFlow YML config using the `bitbucket.token` configuration value, the expected format is `<userid>:<access token>`. 


## <a name="bitbucketserver">Bitbucket Server</a>

### Native Webhooks
Similar to cloud, but requires a shared secret field, which is used to sign/authenticate the request.  The `token` webhook URL parameter used in Bitbucket Cloud is therefore not necessary.

### Post Webhooks
The [Post Webhooks plugin](https://marketplace.atlassian.com/apps/1215474/post-webhooks-for-bitbucket?hosting=server&tab=overview) for Bitbucket server is supported by appending the path `/postwebhook` to the CxFlow webhook URL in the Post Webhooks configuration.

example: `http://cxflow/postwebhook`


Post Webhooks allows global, per-project and per-repository webhook configurations.  The native Bitbucket Server webhook configurations are not as flexible.  Using Config As Code with a single global webhook configuration may be an effective way to deploy CxFlow to your development organization.



The Post Webhooks plugin does not support signature authentication, so one of the following two options must be used to present the shared key to CxFlow:

* Use the `token` URL parameter<br>
example: `http://cxflow/postwebhook?token=XXXXX`<br><br>


* Use the "Basic Authentication" feature in the Post Webhooks configuration.  The username can be omitted from the configuration; the password should be configured to match the `bitbucket.webhook-token` configured value.  Example:

[[/Images/postwebhooks_password.png | Post Webhooks token config]]




