---
title: Set Up Call Parking in Genesys Cloud
author: jason.wolfgang
indextype: blueprint
icon: blueprint
image: images/call-park-workflow.png
category: 6
summary: |
  This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud to park an active voice call with a code and retrieve it using the code.
---
:::{"alert":"primary","title":"About Genesys Cloud Blueprints","autoCollapse":false} 
Genesys Cloud blueprints were built to help you jump-start building an application or integrating with a third-party partner. 
Blueprints are meant to outline how to build and deploy your solutions, not a production-ready turn-key solution.
 
For more details on Genesys Cloud blueprint support and practices 
please see our Genesys Cloud blueprint [FAQ](https://developer.genesys.cloud/blueprints/faq)sheet.
:::

This Genesys Cloud Developer Blueprint explains how to set up Genesys Cloud to park an active voice call with a code and retrieve it using the code.

When an Genesys Cloud user transfers an active call to an in-queue flow with a code, the call can be retrieved from another phone, station or user using the code associated with that call.

![Call Park Genesys Cloud flow](images/call-park-workflow.png "Genesys Cloud Call Park")

The following illustration shows the end-to-end user experience that this solution enables.

![End-to-end user experience](images/CallParkingMid.gif "End-to-end user experience")

:::primary
  **Note:** This example above shows a call being retrieved from a Google Voice number.  A call can also be retrieved by a Genesys Cloud user from the Genesys Cloud platform.  A Genesys Cloud user could also consider sending the retrieval code to the desired Genesys Cloud user via a GC Collaborate Chat or SMS.  An example message could be: "you have a parked call. dial 3172222222,,,,,123 to retrieve the parked call."
  :::

## Solution components

* **Genesys Cloud** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. Contact center agents use the Genesys Cloud user interface.
* **Genesys Cloud API** - A set of RESTful APIs that enables you to extend and customize your Genesys Cloud environment.

## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud
* Experience with Terraform

### Genesys Cloud account

* A Genesys Cloud license. For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the Genesys Cloud pricing page") in the Genesys website.
* The Master Admin role. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.
* CX as Code. For more information, see [CX as Code](https://developer.genesys.cloud/devapps/cx-as-code/ "Goes to the CX as Code page") in the Genesys Cloud Developer Center.

### Development tools running in your local environment

* Terraform (the latest binary). For more information, see [Install Terraform](https://www.terraform.io/downloads.html "Goes to the Install Terraform page") on the Terraform website.


## Implementation steps

You can implement Genesys Cloud objects manually.
* [Configure Genesys Cloud manually](#configure-genesys-cloud-manually)
* [Run Terraform](#run-terraform)



## Run Terraform

### Download the repository containing the project files

Clone the [genesys-cloud-call-park repository](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park "Goes to the genesys-cloud-call-park repository") in GitHub.

### Set up Genesys Cloud

1. You need to set the following environment variables in a terminal window before you can run this project using the Terraform provider:

 * `GENESYSCLOUD_OAUTHCLIENT_ID` - This variable is the Genesys Cloud client credential grant Id that CX as Code executes against. 
 * `GENESYSCLOUD_OAUTHCLIENT_SECRET` - This variable is the Genesys Cloud client credential secret that CX as Code executes against. 
 * `GENESYSCLOUD_REGION` - This variable is the Genesys Cloud region in your organization.

2. Set the environment variables in the folder where Terraform is running. 

### Configure your Terraform build

Set the following values in the **blueprint/terraform/dev.auto.tfvars** file, specific to your Genesys Cloud organization:

* `client_id` - The value of your OAuth Client ID using Client Credentials to be used for the data action integration.
* `client_secret`- The value of your OAuth Client secret using Client Credentials to be used for the data action integration.

The following is an example of the dev.auto.tfvars file.

```
client_id       = "your-client-id"
client_secret   = "your-client-secret"
```

### Run Terraform

The blueprint solution is now ready for your organization to use. 

1. Change to the **/terraform** folder and issue the following commands:

* `terraform init` - This command initializes a working directory containing Terraform configuration files.
  
* `terraform plan` - This command executes a trial run against your Genesys Cloud organization and displays a list of all the Genesys Cloud resources created. Review this list and ensure that you are comfortable with the plan before moving on to the next step.

* `terraform apply --auto-approve` - This command creates and deploys the necessary objects in your Genesys Cloud account. The --auto-approve flag completes the required approval step before the command creates the objects.

Once the `terraform apply --auto-approve` command has completed, you should see the output of the entire run along with the number of objects that Terraform successfully created. The following points should be remembered:

* In this project, assume you are running using a local Terraform backing state. In this case, the `tfstate` files are created in the same folder where the project is running. It is not recommended to use local Terraform backing state files unless you are running from a desktop or are comfortable deleting files.

* As long as you keep your local Terraform backing state projects, you can tear down this blueprint solution by changing to the `docs/terraform` folder. You can also issue a `terraform destroy --auto-approve` command. All objects currently managed by the local Terraform backing state are destroyed by this command.
### Development tools running in your local environment

* Terraform (the latest binary). For more information, see [Install Terraform](https://www.terraform.io/downloads.html "Goes to the Install Terraform page") on the Terraform website.


## Configure Genesys Cloud manually

### Create custom roles for use with Genesys Cloud OAuth clients

Create a custom role for use with a Genesys Cloud OAuth client with the following permissions.

| Roles           | Permissions | Role Name |
|-----------------|-------------------------|---------|
| Custom role 1 | **Conversation** > **All Permissions**, **Analytics** > **Conversation Detail** > **All Permissions**, **Analytics** > **Agent Conversation Detail** > **View**   | Orbit Data Actions OAuth Client |

To create a custom role in Genesys Cloud:

1. Navigate to **Admin** > **Roles/Permissions** and click **Add Role**.

   ![Add a custom role](images/createRole.png "Add a custom role")

2. Enter the **Name** for your custom role.

    ![Name the custom role](images/nameCustomRole.png "Name the custom role")

3. Search and select the required permission for each of the custom role.
   ![Add permissions to the custom role](images/assignPermissionToCustomRole.png "Add permissions to the custom role")
4. Click **Save** to assign the appropriate permission to your custom role.

:::primary
  **Note:** Assign this custom role to your user before creating the Genesys Cloud OAuth client.
  :::

### Create an OAuth client for use with a Genesys Cloud data action integration

To enable a Genesys Cloud data action to make public API requests on behalf of your Genesys Cloud organization, use an OAuth client to configure authentication with Genesys Cloud.

Create an OAuth client for use with the data action integration with the following custom roles.


| OAuth Client   | Custom role | OAuth Client Name |
|----------------|-------------------------------|-------|
| OAuth Client 1 | Orbit Data Actions OAuth Client | Orbit Data Actions OAuth Client |


To create an OAuth Client in Genesys Cloud:

1. Navigate to **Admin** > **Integrations** > **OAuth** and click **Add Client**.

   ![Add an OAuth client](images/2AAddOAuthClient.png "Add an OAuth client")

2. Enter the name for the OAuth client and select **Client Credentials** as the grant type. Click the **Roles** tab and assign the required role for the OAuth client.

     ![Select the custom role and the grant type](images/2BOAuthClientSetup2.png "Select the custom role and the grant type")

3. Click **Save**. Copy the client ID and the client secret values for later use.

   ![Copy the client ID and client secret values](images/2COAuthClientCredentials2.png "Copy the client ID and client secret values")

:::primary
  **Note:** Ensure that you copy the client ID and client secret values for each of the OAuth clients.
  :::

### Add Genesys Cloud data action integration

Add a Genesys cloud data action integration for each OAuth client being used with this blueprint to call the Genesys Cloud public API to:
* Get Waiting Calls in specific Queue based on External Tag
* Replace participant with ANI
* Update External Tag on Conversation

To create a data action integration in Genesys Cloud:

1. Navigate to **Admin** > **Integrations** > **Integrations** and install the **Genesys Cloud Data Actions** integration. For more information, see [About the data actions integrations](https://help.mypurecloud.com/?p=209478 "Opens the About the data actions integrations article") in the Genesys Cloud Resource Center.

   ![Genesys Cloud data actions integration](images/3AGenesysCloudDataActionInstall.png "Genesys Cloud data actions integration")

2. Enter a name for the Genesys Cloud data action, such as Orbit Data Actions OAuth Integration in this blueprint solution.

   ![Rename the data action](images/3BRenameDataAction.png "Rename the data action")

3. On the **Configuration** tab, click **Credentials** and then click **Configure**.

   ![Navigate to the OAuth credentials](images/3CAddOAuthCredentials.png "Navigate to the OAuth credentials")

4. Enter the client ID and client secret that you saved for the Presence Public API [(OAuth Client 1)](#create-oauth-clients-for-use-with-genesys-cloud-data-action-integrations "Goes to the create an OAuth Client section"). Click **OK** and save the data action.

   ![Add OAuth client credentials](images/3DOAuthClientIDandSecret.png "Add OAuth client credentials")

5. Navigate to the Integrations page and set the presence data action integration to **Active**.

   ![Set the data integration to active](images/3ESetToActive.png "Set the data action integration to active")

### Import the Genesys Cloud data actions

Import the following JSON files from the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository:
* `Get-Waiting-Calls-in-specific-Queue-based-on-External-Tag.custom.json`
* `Replace-participant-with-ANI.custom.json`
* `Update-External-Tag-on-Conversation.custom.json`

Import each of data action files and associate with the Orbit Data Actions Oauth Integration data action integration, which uses the Orbit Data Actions OAuth Client.

1. From the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository, download the `Get-Waiting-Calls-in-specific-Queue-based-on-External-Tag.custom.json` file.

2. In Genesys Cloud, navigate to **Integrations** > **Actions** and click **Import**.

   ![Import the data action](images/4AImportDataActions.png "Import the data action")

3. Select the `Get-Waiting-Calls-in-specific-Queue-based-on-External-Tag.custom.json` file and associate with the [Orbit Data Actions Oauth Integration](#add-genesys-cloud-data-action-integrations "Goes to the Add Genesys Cloud data action integrations section") integration, and then click **Import Action**.

   ![Import the Get Waiting Calls in specific Queue based on External Tag data action](images/4BImportGetWaitingCallDataAction.png "Import the Update Genesys Cloud User Presence data action")


4. From the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository, download the `Replace-participant-with-ANI.custom.json` file.

5. In Genesys Cloud, navigate to **Integrations** > **Actions** and click **Import**.

   ![Import the data action](images/4AImportDataActions.png "Import the data action")

6. Select the `Replace-participant-with-ANI.custom.json` file and associate with the [Orbit Data Actions Oauth Integration](#add-genesys-cloud-data-action-integrations "Goes to the Add Genesys Cloud data action integrations section") integration, and then click **Import Action**.

   ![Import the Replace participant with ANI data action](images/4BImportReplaceParticipantAniDataAction.png "Import the Replace participant with ANI data action")

7. From the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository, download the `Update-External-Tag-on-Conversation.custom.json` file.

8. In Genesys Cloud, navigate to **Integrations** > **Actions** and click **Import**.

  ![Import the data action](images/4AImportDataActions.png "Import the data action")

9. Select the `Update-External-Tag-on-Conversation.custom.json` file and associate with the [Orbit Data Actions Oauth Integration](#add-genesys-cloud-data-action-integrations "Goes to the Add Genesys Cloud data action integrations section") integration, and then click **Import Action**.

  ![Import the Update External Tag on Conversation data action](images/4BImportUpdateExternalTagOnConversationDataAction.png "Import the Update External Tag on Conversation data action")

### Import the Architect workflows

This solution includes multiple Architect flows.

* The first two flows, **Default In-Queue Flow** and **Call Park - Agent**, are simple flows that are necessary for the third and final flow.

* The third is an inbound call flow.  The **Orbit - Parked Call Retrieval.i3InboundFlow** flow allows the parked call to be retrieved with the code stored on the External Tag on the conversation record.  It uses the two of the [data actions](#add-genesys-cloud-data-action-integrations "Goes to the Add a web services data actions integration section").

* The last flow is an in-queue call flow.  The **InQueue - Orbit Call Park Hold.i3InQueueFlow** flow plays hold music for the customer has the parked call is waiting to be retrieved.



First import these workflows to your Genesys Cloud organization:

1. Download the `Call Park - Agent Inbound Flow` and `Default In-Queue Flow` files from the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository.

2. In Genesys Cloud, navigate to **Admin** > **Architect** > **Flows: In-Queue Call** and click **Add**.

   ![Import the workflow](images/AddWorkflow2.png "Import the flow")

3. Enter a name for the workflow and click **Create Flow**.

   ![Name your workflow](images/DefaultInQueueFlow.png "Name your flow")

4. From the **Save** menu, click **Import**.

   ![Import the workflow](images/ImportDefaultInQueueFlow.png "Import the workflow")

5. Select the downloaded **Default In-Queue Flow.i3InQueueFlow** file and click **Import**.

6. After importing the In-Queue flow, import the Inbound Call Flow. Proceed to **Admin** > **Architect** > **Flows: Inbound Call** and click **Add**.

 ![Import the workflow](images/AddWorkflow1.png "Import the flow")

7. Enter a name for the workflow and click **Create Flow**.

   ![Name your workflow](images/DefaultAgentInboundFlow.png "Name your flow")

8. From the **Save** menu, click **Import** then, select the downloaded **Call Park - Agent Inbound Flow.i3InQueueFlow** file and click **Import**.

9. Download the `InQueue - Orbit Call Park Hold.i3InQueueFlow` file from the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository.

10. In Genesys Cloud, navigate to **Admin** > **Architect** > **Flows:In-Queue Call** and click **Add**.

   ![Import the workflow](images/AddWorkflow2.png "Import the flow")


11. Enter a name for the workflow and click **Create Flow**.

   ![Name your workflow](images/NameWorkflow2.png "Name your flow")

12. From the **Save** menu, click **Import**.

   ![Import the workflow](images/ImportWorkflow2.png "Import the workflow")

13. Select the downloaded **InQueue - Orbit Call Park Hold.i3InQueueFlow** file and click **Import**.

14. Review your workflow. Click **Save** and then click **Publish**.
![Save your workflow](images/ImportedWorkflow3.png "Save your workflow")

15. Download the `Orbit - Parked Call Retrieval.i3InboundFlow` file from the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository.


16. In Genesys Cloud, navigate to **Admin** > **Architect** > **Flows:Inbound call** and click **Add**.

   ![Import the workflow](images/AddWorkflow1.png "Import the flow")

17. Enter a name for the workflow and click **Create Flow**.

   ![Name your workflow](images/NameWorkflow1.png "Name your flow")

18. From the **Save** menu, click **Import**.

   ![Import the workflow](images/ImportWorkflow1.png "Import the workflow")

19. Select the downloaded **Orbit - Parked Call Retrieval.i3WorkFlow** file and click **Import**.

20. Review your workflow. Map the Data Actions referenced in the flow to the Data Actions with the corresponding names you imported earlier in this blueprint.  For the **Get Waiting Calls in specific Queue based on External Tag** action, make sure the value in the **holdingQueueId** parameter matches the id of **InQueue - Orbit Call Park Hold** in-queue call flow. Click **Save** and then click **Publish**.
![Save your workflow](images/ImportedWorkflow1.png "Save your workflow")

### Create Queues

1. From Admin Home, navigate to Contact Center>Queues and click **Create Queue**

   ![Create Queue](images/CreateQueue.png "Create Queue")

2. Give your Queue a name

  ![Name Queue](images/NameQueue.png "Name Queue")

  `Note: Do not forget to add your self on your created queue`

3. Create another queue, this time for **Call Park - Agent inbound flow** follow the same steps on step 1 and 2. Name the queue as **Call Park Agent Inbound Queue** 

### Import the Script

Create the trigger that invokes the created Architect workflow.

1. From Admin Home, search for **Scripts** and navigate to the Scripts list and click **Import**.

   ![Navigate to Scripts](images/NavigateToScripts.png "Navigate to Scripts")

2. Download the `Orbit Queue Transfer.script` file from the [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) GitHub repository.

3. Select the downloaded **Orbit Queue Transfer.script** file and click **Import**.

   ![Import the Script](images/ImportScript.png "Import the Script")

4. Open your new script and navigate to the **Actions** subtab.

  ![Configure the Script](images/ConfigureScript.png "Configure the Script")

5. From the **Actions** subtab, click "Update Tag and Transfer"

  ![Configure the Script](images/ConfigureScript3.png "Configure the Script")

6. Expand the **Data Actions.Execute Data Action**, section.  Select the category (or data action integration) for your data action, then select the **Update External Tag on Conversation** data action from **Data Action** drop menu.  Expand the input section and map the script variables to the data action inputs.

  ![Configure the Script](images/ConfigureScript2.png "Configure the Script")

7. Expand the **Scripter.Blind Transfer**, section.  Select the Queue where you would like to park the call.  In this example, it is the "Orbit" queue.

  ![Configure the Script](images/SelectBlindTransferQueue.png "Configure the Script")

8. Click **Save** then **Publish** the script.

 
### Add Script to Queue

1. From Admin Home, search for **Queues** and navigate to the Queues list.  Click any inbound Queue you would like your agents to be able to park calls from.

2.  From the queue record, click the **Voice** tab and select this script as the default script for the queue.  Click **Save**

  ![Configure the Script](images/UseScriptInQueue.png "Use the Script")


### Create Call Route for Parked calls

1. Click the **Voice** tab, and define a **Calling Party Number** and **In-queue Flow** for your queue.  This will be the InQueue - Orbit Call Park Hold flow you created in the previous step.

  ![Configure Queue](images/VoiceConfigQueue.png "Configure Queue")

2. From Admin Home, navigate to **Routing>Call Routing** and click **+Add**

  ![Create Call Route](images/CreateCallRoute.png "Create Call Route")

3. Give your Call Route a name, an inbound Number and route it to the **Orbit - Parked Call Retrieval** inbound call flow you created earlier in this blueprint.

  ![Configure Call Route](images/ConfigureCallRoute.png "Configure Call Route")

  4. Repeat steps 1 to 3, and create another call route. Name it **Call Parking Inbound Agent Flow** this time add **Call Park - Agent Inbound Flow** to Route to. 
  
  ![Configure Call Route](images/CallParkingInboundAgentFlowCallRouting.png "Configure Call Route")



  ### Final Steps

  1. Go back to the previously created **Call Park - Agent inbound flow**. Make sure to add the **Call Park Agent Inbound Queue** to the Queue, and add **InQueue - Orbit Call Park Hold to In-Queue** to the Call Flow.

     ![Configure Call Flow](images/ConfigureCallParkAgentInboundFlow.png "Configure Call Flow")
   
   2. Make sure to set the **holidingQueueId** same as the id of **Orbit** queue, the first queue we created earlier.  

   ![Configure Call Flow](images/Orbit-ParkedCallRetrievalQueueId.png "Configure Call Flow")


## Additional resources

* [Genesys Cloud API Explorer](https://developer.genesys.cloud/devapps/api-explorer "Opens the GC API Explorer") in the Genesys Cloud Developer Center
* The [genesys-cloud-call-park repo](https://github.com/GenesysCloudBlueprints/genesys-cloud-call-park) repository in GitHub