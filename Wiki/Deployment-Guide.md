To begin, you will need:

* An Azure subscription where you can create the following kind of resources:

* App service

* App service plan

* Bot channels registration

* Azure storage account

* Azure search

* Application Insights

* A copy of the Workplace Awards app GitHub [repo](https://github.com/OfficeDev/microsoft-teams-apps-workplaceawards)

## Step 1: Register Azure AD applications

Register one Azure AD applications in your tenant's directory: for the bot and tab app authentication.

  
1. Log in to the Azure Portal for your subscription, and go to the "App registrations" blade [here](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps).

2. Click on "New registration", and create an Azure AD application.

3.  **Name**: The name of your Teams app - if you are following the template for a default deployment, we recommend "Workplace Awards".

4.  **Supported account types**: Select "Accounts in any organizational directory"

5. Leave the "Redirect URL" field blank.

    [[/Images/multitenant_app_creation.PNG|Multi tenant selection in app registration portal]]

6. Click on the "Register" button.

7. When the app is registered, you'll be taken to the app's "Overview" page. Copy the **Application (client) ID**; we will need it later. Verify that the "Supported account types" is set to **Multiple organizations**.

    [[/Images/MultitenantAppOverview.PNG| App registration overview page]] 

8. On the side rail in the Manage section, navigate to the "Certificates & secrets" section. In the Client secrets section, click on "+ New client secret". Add a description for the secret and select Expires as "Never". Click "Add".

    [[/Images/multitenant_app_secret.PNG| App registration secret overview page]]

9. Once the client secret is created, copy its **Value**, please take a note of the secret as it will be required later.

At this point you have 3 unique values:
 
* Application (client) ID which will be later used during ARM deployment as Bot Client id and in manifest files as `<<botid>>`

* Client secret for the bot which will be later used during ARM deployment as Bot Client secret

* Directory (tenant) ID
 
We recommend that you copy these values into a text file, using an application like Notepad. We will need these values later. 

## Step 2: Deploy to your Azure subscription

1. Click on the "Deploy to Azure" button below.

[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FOfficeDev%2Fmicrosoft-teams-apps-workplaceawards%2Fmaster%2FDeployment%2Fazuredeploy.json) 

2. When prompted, log in to your Azure subscription.

3. Azure will create a "Custom deployment" based on the ARM template and ask you to fill in the template parameters.

    [[/Images/CustomDeployment.PNG|Custom deployment page]]

4. Select a subscription and resource group.

    * We recommend creating a new resource group.

    * The resource group location MUST be in a data center that supports: Application Insights; and Azure Search. For an up-to-date list, click [here](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=logic-apps,cognitive-services,search,monitor), and select a region where the following services are available:

    * Application Insights

    * Azure Search

5. Enter a "Base Resource Name", which the template uses to generate names for the other resources.

    * The app service names `[Base Resource Name]`, must be available. For example, if you select `contosoworkplaceawards` as the base name, the names `contosoworkplaceawards` must be available (not taken); otherwise, the deployment will fail with a Conflict error.

    * Remember the base resource name that you selected. We will need it later.

6. Fill in the various IDs in the template:

    * **Bot Client ID**: The application (client) ID registered in Step 1. 

    * **Bot Client Secret**: The client secret registered in Step 1.   

    * **Tenant Id**: The tenant ID registered in Step 1. If your Microsoft Teams tenant is same as Azure subscription tenant, then we would recommend to keep the default values. 
  
    Make sure that the values are copied as-is, with no extra spaces. The template checks that GUIDs are exactly 36 characters.

7. If you wish to change the app name, description, and icon from the defaults, modify the corresponding template parameters.

> NOTE: If you plan to use a custom domain name instead of relying on Azure Front Door, read the instructions [here](https://github.com/OfficeDev/microsoft-teams-apps-workplaceawards/wiki/Custom-domain-option) first.

8. Click on "Review+Create" to start the deployment.It will validate the parameters provided in the template .Once the validation is passed,click on create to start the deployment.

   [[/Images/Validationpassed.png|Deploy to Azure]]

9. Wait for the deployment to finish. You can check the progress of the deployment from the "Notifications" pane of the Azure Portal. It can take more than 20 minutes for the deployment to finish.

## Step 3: Set up authentication for the app

1. Go back to the "App Registrations" page [here](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)

1. Enter the Bot client id created in Step 1 under Owned applications search box.

1. Click on the application (this should be the same application registered in step 1)

1. Under left menu, select **Authentication** under **Manage** section. 

1. Select 'Accounts in any organizational directory (Any Azure AD directory - Multitenant)' under Supported account types and click "+Add a platform". 


1. On the flyout menu, Select "Web"

    [[/Images/RedirectUrlMenu.png|Redirect web URL selection flyout menu]]

1. Add `**Redirect URI:** Enter `%redirectUri%` from the template deployment output for the URL e.g. `https://[baseresourcename].azurefd.net/signin-simple-end`and select the check boxes "Access tokens" and "ID tokens"and then click "Configure" button and the bottom. 
   For e.g. 

   [[/Images/Configureweb.png|Set up authentication for the app]]

1. Under left menu, select  **Expose an API**  under  **Manage**. 

    [[/Images/ExposeAnApiMenu.png|Expose an API menu]]

1. Select the  **Set**  link to generate the Application ID URI in the form of  `api://{BotID}`. Insert your fully qualified domain name (with a forward slash "/" appended to the end) between the double forward slashes and the GUID. The entire ID should have the form of:  `api://fully-qualified-domain-name.com/{BotID}`
    -   for e.g.:  `api://subdomain.example.com:6789/c6c1f32b-5e55-4997-881a-753cc1d563b7`.

1.  Select the  **Add a scope**  button. In the panel that opens, enter  `access_as_user`  as the  **Scope name**.

1.  Set Who can consent? to "Admins and users"

1.  Fill in the fields for configuring the admin and user consent prompts with values that are appropriate for the  `access_as_user`  scope. Suggestions:
    -   **Admin consent display name:**  Workplace awards
    -   **Admin consent description**: Allows Teams to call the app’s web APIs as the current user.
    -   **User consent display name**: Teams can access your user profile and make requests on your behalf
    -   **User consent description:**  Enable Teams to call this app’s APIs with the same rights that you have

1.  Ensure that  **State**  is set to  **Enabled**

1.  Select  **Add scope**
    -   Note: The domain part of the  **Scope name**  displayed just below the text field should automatically match the  Application ID URI set in 
        the previous step, with  `/access_as_user`  appended to the end; for example:
        -   `api://subdomain.example.com:6789/c6c1f32b-5e55-4997-881a-753cc1d563b7/access_as_user`

1.  In the same page in below section **Authorized client applications**, you identify the applications that you want to authorize to your app’s web application. Each of the following IDs needs to be entered. Click "+Add a client application" and copy-paste the below id and select checkbox "Authorized scopes". Repeat the step for second GUID. 
   -   `1fec8e78-bce4-4aaf-ab1b-5451cc387264`  (Teams mobile/desktop application)
   -   `5e3ce6c0-2b1f-4285-8d4b-75ee78787346`  (Teams web application)

1.  Under left menu, navigate to  **API Permissions**, and make sure to add the follow permissions of Microsoft Graph API > Delegated permissions:
   -   User.Read (enabled by default)
   -   email
   -   offline_access
   -   openid
   -   profile

**Note:** The detailed guidelines for registering an application for SSO Microsoft Teams tab can be found [here](https://docs.microsoft.com/en-us/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso)

## Step 4: Create the Teams app packages

This step covers the Teams application package creation for teams scope and make it ready to install in Teams. 

1. Open the `Manifest\manifest.json` file in a text editor.

1. Change the placeholder fields in the manifest to values appropriate for your organization.

   *  `developer.name` ([What's this?](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#developer))

   *  `developer.websiteUrl`

   *  `developer.privacyUrl`

   *  `developer.termsOfUseUrl`

1. Change the `<<botId>>` placeholder to your Azure AD application's ID from above. This is the same GUID that you entered in the template under "Bot Client ID".

1. Replace `<appbaseurl>` placeholder in configurationTabs.configurableUrl with the Azure front door URL created i.e.'https://[BaseResourceName].azurefd.net'. For example if you chose contoso-workplaceawards then the appbaseurl will be https://contoso-workplaceawards.azurefd.net. 
  > note : please make sure to keep the URL path `/config-tab` as is. 

1. In the "validDomains" section, replace the `<<appDomain>>` with your Bot App Service's domain. This will be `[BaseResourceName].azurefd.net`. For example if you chose "contosoworkplaceawards" as the base name, change the placeholder to `contosoworkplaceawards.azurefd.net`.
  > note : please make sure to not add https:// in valid domains.

1. In the "webApplicationInfo" section, replace the `<<botId>>` with Bot Client ID of the app created in Step 1. 

1. Replace `<<ApplicationIdURI>>` with the Application ID URI value of your registered app. This will have a value in a format similar to `api://<<applicationurl>>/<<botId>>`. For example `api://contoso-workplaceawards.azurefd.net/19c1102a-fffe-46c4-9a85-016bec13e0ab` where contoso-workplaceawards is the base resource URL used under valid domains and configurable tabs and 19c1102a-fffe-46c4-9a85-016bec13e0ab is the bot client id. 

1. Create a ZIP package with the `manifest.json`,`color.png`, and `outline.png`. The two image files are the icons for your app in Teams.

* Make sure that the 3 files are the _top level_ of the ZIP package, with no nested folders.
 
  [[/Images/ManifestUI.png|Manifest file in windows explorer]]

## Step 5: Run the apps in Microsoft Teams

1. If your tenant has side loading apps enabled, you can install your app by following the instructions
[here](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/apps/apps-upload#load-your-package-into-teams)

1. You can also upload it to your tenant's app catalog, so that it can be available for everyone in your tenant to install. See [here](https://docs.microsoft.com/en-us/microsoftteams/tenant-apps-catalog-teams)

    * We recommend using [app permission policies](https://docs.microsoft.com/en-us/microsoftteams/teams-app-permission-policies) to restrict access to this app to the members of the experts team.

1. Install the app (the `WorkplaceAwards.zip` package) to your team.

### Troubleshooting

Please see our [Troubleshooting](https://github.com/OfficeDev/microsoft-teams-apps-workplaceawards/wiki/Troubleshooting) page.