# Azure Migrate application credential assessment and remediation guidance

## Disclaimer

Guidance in this document applies only in relation to the mitigation steps necessary for the issue disclosed in the [CVE](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42306) and detailed in [Microsoft Security Response Center blog](https://aka.ms/CVE-2021-42306-AAD). Do not use this guidance as general credential rotation procedure.

## What actions am I required to take?

Azure Migrate appliances that were registered after **11/02/2021 9:55 AM UTC** and had [Appliance configuration manager version 6.1.220.1 and above](https://docs.microsoft.com/azure/migrate/migrate-appliance#check-the-appliance-services-version) are **not** impacted and do **not** require further action.

For Azure Migrate appliances registered prior to 11/02/2021 9:55 AM UTC or appliances where [auto-update was disabled](https://docs.microsoft.com/azure/migrate/migrate-appliance#appliance-upgrades) and registered after 11/02/2021 9:55 AM UTC, we recommend you execute the assessment script mentioned below to identify any impacted Azure AD applications and then perform the mitigation steps.

## Assessment script

The assessment script can help identify any impacted Azure AD applications associated with Azure Migrate within the tenant. Before executing the script, make sure you have met the following prerequisites:

> [!NOTE]
>  You can run the script from any Windows server with internet connectivity.

### Assessment script prerequisites

1. Windows PowerShell version 5.1 or later installed.
1. .NET framework 4.7.2 or later installed.
1. The following URLs are accessible from the server:
   **Azure public cloud URLs**  
    - *.powershellgallery.com
    - login.microsoftonline.com
    - graph.windows.net
    - management.azure.com
    - *.azureedge.net
    - aadcdn.msftauth.net
    - aadcdn.msftauthimages.net
    - dc.services.visualstudio.com
    - aka.ms/*
    - download.microsoft.com/download
    - go.microsoft.com/*

   > [!NOTE]
   >  If there is a proxy server blocking access to these URLs, then update the proxy details to the system configuration before executing the script.

4. You need the following permissions on the Azure user account:

-  **Application.Read** permission at tenant level to enumerate the impacted Azure AD applications.
-  **Contributor** access on the Azure subscription(s) to enumerate the Azure Migrate resources associated with the impacted Azure AD applications.

### Execution instructions

1. Log in to any Windows server with the internet connectivity.
1. Download the .zip file with [assessment script](https://aka.ms/azuremigrateimpactassessment) on the server.
1. Extract the contents from the .zip file and open Windows PowerShell as an Administrator.
1. Change the folder path to the location where the file was extracted.
1. Execute the script by running the following command _(see how to find the Tenant ID [here](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-how-to-find-tenant))_:

   `.\AssessAzMigrateApps.ps1 -TenantId <provide your tenant ID> -ScanAll`

   > [!NOTE]
   >  Running the script with -ScanAll parameter across the tenant may take time to execute depending on the number of Azure AD applications in your tenant.

1. When prompted, log in to your Azure user account. The user account should have permissions listed in prerequisites above.
1. The script generates an assessment report with the details of the impacted Azure AD applications and associated Azure Migrate resources.

### What the assessments script does?

1. Connects to the tenant ID provided in the command using the Azure account; user provides to log in through the script.
1. Scans and finds all the impacted Azure AD applications with the unprotected private key.
1. Identifies the impacted Azure AD applications, associated with Azure Migrate.
1. Finds the Azure Migrate resources accessible to the currently logged in user across subscriptions within the tenant.
1. Maps the impacted Azure Migrate resources information to the impacted Azure AD applications found in Step 3.
1. Generates an assessment report with the information of the impacted Azure AD applications with the details of the associated Azure Migrate resources.

### Assessment report

The assessment report generated by the script contains the following columns:

**No** | **Column name** | **Description** |
--- | --- | ---|
1 |Azure AD application name | The names of the impacted Azure AD applications associated with Azure Migrate, containing one of the following suffixes: </br> -  resourceaccessaadapp </br> - agentauthaadapp </br>  -  authandaccessaadapp
2 |Azure AD application owner | Provides the email address of the Azure AD application owner.
3 |Azure AD application ID | The ID of the impacted Azure AD applications.
4 |User access to associated Migrate resources | Shows if the currently logged- in user has access to the associated Azure Migrate resources across subscriptions in the tenant.
5 |Subscription ID | The IDs of subscriptions where the currently logged in user could access the Azure Migrate resources.
6 |Resource Group | The name of the Resource Group where the Azure Migrate resources were created.
7 |Azure Migrate project name | The name of the Azure Migrate project where the Azure Migrate appliance(s) were registered.
8 |Azure Migrate appliance name | The name of the Azure Migrate appliance which created the impacted Azure AD application during its registration.
9 |Scenario | The scenario of the appliance deployed-VMware/Hyper-V/Physical or other clouds.
10 |Appliance activity status (last 30 days) | The information on whether the appliance was active in the last 30 days *(agents sent heartbeat to Azure Migrate services)*
11 |Appliance server hostname | The hostname of the server where the appliance was deployed.</br> *(This may have changed over time in your on-premises environment)*

### Recommendations

Based on the information you have from the script in the context of the currently logged in user. We recommend you take one of the following actions:

1. For rows, where column 4 (User access to associated Migrate resources) shows *Not accessible*, it can be due to one of the following reasons:
   - You do not have the required permissions _(as stated in prerequisites above)_ on the subscription to enumerate information for Azure Migrate resources associated with the impacted Azure AD applications.
   - The Azure Migrate resources associated with the impacted Azure AD application may have been deleted.
1. For cases where you are sure that the Azure Migrate resources have been deleted or there are inactive Azure Migrate appliances that you do not intend to use in future, you can go to the impacted Azure AD application on Azure portal, select 'Certificates & secrets' and delete all the certificates associated with the application.
1. For active Azure Migrate appliances that you intend to use in future, you need to rotate the certificates on the impacted Azure AD applications using the mitigation script on each of the appliance servers _(server hostname provided in the assessment report)_.

## Mitigation script

After assessing the impacted Azure AD applications, you need to **execute the mitigation script on each Azure Migrate appliance in your organization's environment**. You can check if you have Azure Migrate appliances in your environment from [here](https://docs.microsoft.com/azure/migrate/common-questions-appliance#how-do-i-find-the-azure-migrate-appliances-registered-to-the-project). Before executing the script, ensure you have met the following prerequisites:

### Mitigation script prerequisites

1. Windows PowerShell version 5.1 or later installed
1. Windows PowerShell running 64-bit version
1. .NET framework 4.7.2 or later installed.
1. The following URLs accessible from the server (In addition to the other [URLs](https://docs.microsoft.com/azure/migrate/migrate-appliance#public-cloud-urls) you have already whitelisted for the appliance registration):
   **Azure public cloud URLs
   -  *.powershellgallery.com
   -  *.azureedge.net
   -  aadcdn.msftauthimages.net

   > [!NOTE]
   > If there is a proxy server configured on the appliance configuration manager, then update the proxy details to the system configuration before executing the script.*

1. You need the following permissions on the Azure user account:
   -  'Contributor’ access on the Azure subscription that has the Azure Migrate project the appliance is registered to.
   -  Owner permissions on the impacted Azure AD Application(s).
1. For appliances deployed to perform agentless replication of VMware VMs, if you have started replication for the first time in the project from Azure portal, we recommend you to wait for five minutes for the *Associate replication policy* job to complete before you can execute the script.

### Run the script

1. To run the script, you need to log in as an Administrator on the server hosting the Azure Migrate appliance.
1. Download the .zip file with [mitigation script](https://aka.ms/azuremigratecertrotation) on the appliance server.
1. Extract the contents from the .zip file and open Windows PowerShell as an Administrator.
1. Change the folder path to the location where the file was extracted.
1. Execute the script by running the following command:

   `.\AzureMigrateRotateCertificate.ps1`
1. When prompted, log in to your Azure user account. The user account should have permissions listed in prerequisites.
1. Wait for the script to execute success.

> [!NOTE]
>  After executing the mitigation script in the appliance, you can verify if the issue has been resolved by running the assessment script again using the following command:
`.\AssessAzMigrateApps.ps1 -TenantId <provide your tenant ID> -AppId <provide the impacted application ID>`. The script should return 0 applications impacted if the mitigation ran successfully.

### What the mitigation script does?

1. Fetches Key Vault name, certificate name and Azure AD App ID from Azure Migrate Hub/configuration files on appliance server.
1. Rotates the certificate in the Key Vault with new private public key pair.
1. Imports the new certificate to appliance server in PFX format.
1. Deletes the old certificate in the impacted Azure AD App that removes the vulnerability of the private key misuse.
1. Attaches the public key of the new certificate in CER format (that was generated in Step 2) to the Azure AD application.
1. Updates the software configuration files on appliance server to use the new certificate and restarts the Azure Migrate appliance agents.

### Request for support

If case of any issues during the execution of the above scripts, request assistance from Microsoft support. While creating support request follow the steps below:

1.	In the 'Summary' field, provide the details as follows: "CVE-2021-42306: \<Issue summary>"
1. Problem type: “Discovery and assessment”
1. Problem subtype: _any one of the following_
   -  Deployment issues with Azure Migrate appliance for VMware
   -  Deployment issues with Azure Migrate appliance for Hyper-V
   -  Deployment issues with Azure Migrate appliance for Physical
1. In the 'Additional details' section include whether issue is related to **Assessment script** or **Mitigation script**.
1. In case of an issue with script execution, please share the screenshot of the error.


