## Prerequisites

To begin, you will need: 

* Two Azure subscriptions attached to two different Office 365 Tenants:
  * Azure Commerical Subscription attached to an Office 365 GCC tenant (user directory)
  	* Bot channels registration
	* Application Insights
  * Azure Government (MAG) Subscription attached to Office 365 GCC High tenant (all other services)
	* App service
	* App service plan
	* Azure storage account
	* Azure search
	* Azure function
	* QnA Maker cognitive service
	* Application Insights  
* A team in Microsoft Teams GCC with your group of experts. (You can add and remove team members later!)
* A copy of the FAQ Plus app repo (this directory)
* A reasonable set of Question and Answer pairs to set up the knowledge base for the bot.
* A copy of the FAQ Plus app GitHub repo (https://github.com/OfficeDev/microsoft-teams-apps-faqplus-hybrid)
* Latest version of the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli)


## Step 1: Register Azure AD applications

Register two Azure AD applications in your GCC tenant's directory: one for the bot, and another for the configuration app.

1. Log in to the Azure Portal for your Azure Commerical subscription, and go to the "App registrations" blade [here](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps).

2. Click on "New registration", and create an Azure AD application.
	1. **Name**: The name of your Teams app - if you are following the template for a default deployment, we recommend "FAQ Plus".
	2. **Supported account types**: Select "Accounts in any organizational directory"
	3. Leave the "Redirect URL" field blank.

![Azure registration page](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/multitenant_app_creation.png)

3. Click on the "Register" button.

4. When the app is registered, you'll be taken to the app's "Overview" page. Copy the **Application (client) ID** and **Directory (tenant) ID**; we will need it later. Verify that the "Supported account types" is set to **Multiple organizations**.

![Azure overview page](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/multitenant_app_overview.png)

5. On the side rail in the Manage section, navigate to the "Certificates & secrets" section. In the Client secrets section, click on "+ New client secret". Add a description of the secret and select an expiry time. Click "Add".

![Azure AD overview page](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/multitenant_app_secret.png)

6. Once the client secret is created, copy its **Value**; we will need it later.

7. Go back to “App registrations”, then repeat steps 2-3 to create another Azure AD application for the configuration app.
	1. **Name**: The name of your configuration app. We advise appending “Configuration” to the name of this app; for example, “FAQ Plus Configuration”.
	2. **Supported account types**: Select "Account in this organizational directory only"
	3. Leave the "Redirect URL" field blank for now.

At this point you have 4 unique values:

* Application (botclientId) ID for the bot
* Client secret for the bot
* Application (configAppClientId) ID for the configuration app
* Directory (tenant) ID, which is the same for both apps

We recommend that you copy these values into a text file, using an application like Notepad. We will need these values later.

![Configuration step 3](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/azure-config-app-step3.png)

## Step 2: Deploy bots to your Commerical Azure subscription
1. Open the Azure CLI.
2. Change the working directory to ~/Deployment so that you have access to the arm templates for deployment.
3. Login to Azure:
	- az cloud set --name AzureCloud
	- az login
		- A webpage will launch allowing you to enter your Azure Commercial credentials. Return to the CLI after authentication is successful.
4. Create a new resource group for your app:
	- az group create --location **desiredregion** --name **resourcegroupname**
5. Deploy the resources:
	- az deployment group create --resource-group **resourcegroupname** --template-file azuredeploy.json
		- You will be prompted for the following:
			- baseResourceName - this will be the prefix of your app, for example: faqplus2021
			- botClientId - This is the client ID from the bot app registration created in Step 1.

[![Deploy To Azure Commercial](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FOfficeDev%2Fmicrosoft-teams-apps-faqplus-hybrid%2Fmaster%2FDeployment%2Fazuredeploy.json)

## Step 3: Deploy remaining resources to your Azure Government subscription
1. Open a new instance of Azure CLI.
2. Change the working directory to ~/Deployment so that you have access to the arm templates for deployment.
3. Login to Azure:
	- az cloud set --name AzureUSGovernment
	- az login
		- A webpage will launch allowing you to enter your Azure Government credentials. Return to the CLI after authentication is successful. 
		- **Note:** It may be easier to copy the launched URL into a private page to avoid your browser inheriting the Azure Commerical login.
4. Create a new resource group for your app:
	- az group create --location **desiredregion** --name **resourcegroupname**
5. Deploy the resources:
	- az deployment group create --resource-group **resourcegroupname** --template-file azuredeploy-gcc-h.json
		- You will be prompted for the following:
			- baseResourceName - this will be the prefix of your app, for example: faqplus2021
			- botClientId - This is the client ID from the bot app registration created in Step 1.
			- botClientSecret - This is the client secret from the bot app registration created in Step 1.
			- botAppInsightsKey - This key can be retrieved from the App Insights deployment from Step 2.
			- configAppClientId - This is the client ID from the bot app registration created in Step 1.
			- configAdminUPNList - This list of email accounts from your GCC tenant that have access to the config website.
				- For example, to allow Megan Bowen (meganb@contoso.com) and Adele Vance (adelev@contoso.com) to access the configuration app, set this parameter to `meganb@contoso.com;adelv@contoso.com`.
* You can change this list later by going to the configuration app service's "Configuration" blade.
			- tenantId - This is the tenant ID from the bot app registration created in Step 1 in the Office 365 GCC tenant

[![Deploy To Azure Gov](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FOfficeDev%2Fmicrosoft-teams-apps-faqplus-hybrid%2Fmaster%2FDeployment%2Fazuredeploy-gcc-h.json)

## Step 3: Set up authentication for the configuration app

1. Note the location of the configuration app that you deployed, which is `https://[BaseResourceName]-config.azurewebsites.us`. For example, if you chose "contosofaqplus" as the base name, the configuration app will be at `https://contosofaqplus-config.azurewebsites.us`

2. Go back to the "App Registrations" page [here](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview).

3. Click on the configuration app in the application list. Under "Manage", click on "Authentication" to bring up authentication settings.

4. Click on Add a platform, select Web.

![Adding Redirect URI1](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/AuthenticationImage1.png)

5. Add new entry to "Redirect URIs":
	If your configuration app's URL is https://contosofaqplus-config.azurewebsites.us, then add the following entry as the Redirect URIs:
	- https://contosofaqplus-config.azurewebsites.us

	Note: Please refer to Step 3.1 for more details about the URL. 

6. Under "Implicit grant", check "ID tokens" and "Access tokens". The reason to check "ID tokens" is because you are using only the accounts on your current Azure tenant and using that to authenticate yourself in the configuration app. Click configure.

![Adding Redirect URI2](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/AuthenticationImage2.png)

7. Add new entries to "Redirect URIs":
	If your configuration app's URL is https://contosofaqplus-config.azurewebsites.us, then add the following entry as the Redirect URIs:
	- https://contosofaqplus-config.azurewebsites.us/signin
	- https://contosofaqplus-config.azurewebsites.us/configuration

![Adding Redirect URI3](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/AuthenticationImage3.png)

8. Click "Save" to commit your changes.

## Step 4: Create the QnA Maker knowledge base

Create a knowledge base on the [QnA Maker Azure Gov portal](https://www.qnamaker.azure.us/Create), following the instructions in the QnA Maker documentation [QnA Maker documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/tutorials/create-publish-query-in-portal#create-a-knowledge-base).

Skip the step, "Create a QnA service in Microsoft Azure", because the ARM template that you deployed in Step 2 "Deploy to your Azure subscription" already created the QnA service. Proceed directly to the next step, "Connect your QnA service to your KB".

Use the following values when connecting to the QnA service:

* **Microsoft Azure Directory ID**: The tenant associated with the Azure subscription used for Step 3.
* **Azure subscription name**: The Azure subscription to which the ARM template was deployed in Step 3.
* **Azure QnA service**: The QnA service was created during the deployment. This is the same as the "Base resource name"; for example, if you chose "contosofaqplus" as the base name, the QnA Maker service will be named `contosofaqplus`.

![Screenshot of settings](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/media/qnamaker-tutorial-create-publish-query-in-portal/create-kb-step-2.png)

### Multi-Turn Enablement
With the new updates to the FAQ Plus app template, the knowledge base can now support multi-turn conversations. To understand the basics of multi-turn conversations, navigate to the [QnA Maker documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/QnAMaker/how-to/multiturn-conversation#what-is-a-multi-turn-conversation) to understand about multi-turn conversations. To enable multi-turn on the newly created knowledge base, go to this [link](https://docs.microsoft.com/en-us/azure/cognitive-services/QnAMaker/how-to/multiturn-conversation#create-a-multi-turn-conversation-from-a-documents-structure) to enable multi-turn extraction. 

* Note: For best practices with regards to formatting and document structure, please follow the guidelines [here](https://docs.microsoft.com/en-us/azure/cognitive-services/QnAMaker/how-to/multiturn-conversation#building-your-own-multi-turn-document).

After [publishing the knowledge base](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/tutorials/create-publish-query-in-portal#publish-to-get-knowledge-base-endpoints), note the knowledge base ID (see screenshot).

![Screenshot of the publishing page](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/kb_publishing.png)

Remember the knowledge base ID: we will need it in the next step.

## Step 5: Finish configuring the FAQ Plus app

1. Go to the configuration app, which is at `https://[BaseResourceName]-config.azurewebsites.us`. For example, if you chose “contosofaqplus” as the base name, the configuration app will be at `https://contosofaqplus-config.azurewebsites.us`.

2. You will be prompted to log in with your credentials. Make sure that you log in with an account that is in the list of users allowed to access the configuration app.

![Config web app page](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/config-web-app-login.png)

3. Get the link to the team with your experts from the Teams client. To do so, open Microsoft Teams, and navigate to the team. Click on the "..." next to the team name, then select "Get link to team".

![Get link to Team](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/get-link-to-Team.png)

Click on "Copy" to copy the link to the clipboard.

![Link to team](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/link-to-team.png)

4. Paste the copied link into the "Team Id" field, then press "OK".

![Add team link form](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/fill-in-team-link.png)

5. Enter the QnA Maker knowledge base ID into the "Knowledge base ID" field, then press "OK".

6. Customize the "Welcome message" that's sent to your End-users when they install the app. This message supports basic markdown, such as bold, italics, bulleted lists, numbered lists, and hyperlinks. See [here](https://docs.microsoft.com/en-us/adaptive-cards/authoring-cards/text-features#markdown) for complete details on what Markdown features are supported.

### Notes

Remember to click on "OK" after changing a setting. To edit the setting later, click on "Edit" to make the text box editable.

## Step 6: Create the Teams app packages

Create two Teams app packages: one for end-users to install personally, and one to be installed to the experts' team.

1. Open the `Manifest\manifest_enduser.json` file in a text editor.

2. Change the placeholder fields in the manifest to values appropriate for your organization.

* `developer.name` ([What's this?](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#developer))

* `developer.websiteUrl`

* `developer.privacyUrl`

* `developer.termsOfUseUrl`

3. Replace all the occurrences of `<<botId>>` placeholder to your Azure AD application's ID from above. This is the same GUID that you entered in the template under "Bot Client ID".

4. In the "validDomains" section, replace all the occurrences of `<<appDomain>>` with your Bot App Service's domain. This will be `[BaseResourceName].azurewebsites.us`. For example, if you chose "contosofaqplus" as the base name, change the placeholder to `contosofaqplus.azurewebsites.us`.

5. Save and Rename `manifest_enduser.json` file to a file named `manifest.json`.

6. Create a ZIP package with the `manifest.json`,`color.png`, and `outline.png`. The two image files are the icons for your app in Teams.
* Name this package `faqplus-enduser.zip`, so you know that this is the app for end-users.
* Make sure that the 3 files are the _top level_ of the ZIP package, with no nested folders.
![File Explorer](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/file-explorer.png)

7. Rename the `manifest.json` file to `manifest_enduser.json` for reusing the file.

8.  Open the `Manifest\manifest_sme.json` file in a text editor.

9. Repeat the steps from 2 to 4 to replace all the placeholders in the file.

10. Save and Rename `manifest_sme.json` file to a file named `manifest.json`.

11. Create a ZIP package with the `manifest.json`,`color.png`, and `outline.png`. The two image files are the icons for your app in Teams.
* Name this package `faqplus-experts.zip`, so you know that this is the app for sme/experts.
* Make sure that the 3 files are the _top level_ of the ZIP package, with no nested folders.
![File Explorer](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Images/file-explorer.png)

12. Rename the `manifest.json` file to `manifest_sme.json` for reusing the file.

## Step 7: Run the apps in Microsoft Teams

1. If your tenant has sideloading apps enabled, you can install your app by following the instructions [here](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/apps/apps-upload#load-your-package-into-teams)

2. You can also upload it to your tenant's app catalog so that it can be available for everyone in your tenant to install. See [here](https://docs.microsoft.com/en-us/microsoftteams/tenant-apps-catalog-teams)

3. Install the experts' app (the `faqplus-experts.zip` package) to your team of subject-matter experts. This **MUST** be the same team that you selected in Step 5.3 above.

    **NOTE:** Do NOT use app permission policies to restrict the experts' app to the members of the subject matter experts team. Teams does not support applying different policies to the same bot via two different app packages. If you do this, you may find that the end-user app does not respond to some users.

4. Install the end-user app (the `faqplus-enduser.zip` package) to your users.

## Troubleshooting

Please see our [Troubleshooting](https://github.com/OfficeDev/microsoft-teams-apps-faqplus/wiki/Troubleshooting) page.