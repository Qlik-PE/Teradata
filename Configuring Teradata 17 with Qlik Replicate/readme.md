![](./media/image1.png)

# **_Configuring Azure ADLS Gen2 with Qlik Data Integration Suite_**

## **Partner Engineering**

<br>
<br>
<br>
<br>

John Park<br>
Principal Solution Architect<br>
john.park@qlik.com<br>
![](./media/image2.png)  

**Version: 1.2**<br>
**Initial Release Date: 17-Feb-20**




**Revisions**      | **Notes**   | **Date**  | **Version**
------------------ | ----------- | --------- | -----------
Initial Draft      | 20-Feb-2020 | John Park | 0.1         |
Review of Language | 20-Feb-2020 | John Park | 0.2         |
Final Edit for V1  | 21-Feb-2020 | John Park | 1.0         |
Edits for Markdown | 25-Feb-2020 | John Park | 1.1         |
Edits for Release  | 27-Feb-2020 | John Park | 1.2        |

# Table of Contents
--------------------

[**Summary**](#summary)

[**Configuring Azure Components**](#configuring-azure-components)

[**Part 1 - Create or Configure Azure Data Lake Storage (ADLS) Gen2**](#part-1---create-or-configure-azure-data-lake-storage-adls-gen2)

[**Part 2 - Create an Active Directory "App Registration"**](#part-2---create-an-active-directory-and-perform-app-registration)

[**Part 3 - Finalizing Configuration of the ADLS-2 Storage Account**](#part-3---finalizing-configuration-of-the-adls-2-storage-account)

[**Part 4 - Create a Target Directory in the File System**](#part-4---create-a-target-directory-in-the-file-system)

## **Summary**
--------------

This document was created to supplement Qlik Replicate and Qlik Compose Documentation. This guide is for customers intending to use Qlik Data Integration and Azure ADLS Gen2 and Azure Databricks. The official documentation can be found at

- <https://help.qlik.com/en-US/compose/Content/Compose/Home.htm>

- <https://help.qlik.com/en-US/replicate/Content/Replicate/Home.htm>

Azure ADLSv2 is foundation piece of Azure Technology and is the Object Store underpinning Azure Data Lake service.

Please see Official Documentation about ADLSv2 here:

- <https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction>

## **Configuring Azure Components**

**High Level Overview**

- Create or Configure Azure Data Lake Storage (ADLS) Gen2

- Create Azure Active Directory Application

- Create ADLS File System and Folder

### **Part 1 - Create or Configure Azure Data Lake Storage (ADLS) Gen2**

First, we need to set up the Azure storage account that Qlik Replicate will use to map data into Azure Databricks. We will setup Azure Data Lake Storage (ADLS Gen2) to manage the external tables.

If you do not already have an ADLS Gen2 storage account, we need to create one now. From your Azure portal home page, click on Storage Accounts.<br>
***Figure A.1.0.***

![](./media/image3.png)

Inside Storage Accounts click "+ Add".<br>
***Figure A.1.1***

![](./media/image4.png)

Select your Azure subscription and an existing resource group or select "Add" to create a new storage account. Next choose a name for your storage account and a location. For "Account kind"be sure that it says Storage V2 (general purpose v2).Click "Next: Networking" at the bottom of the page. Make note of the name of the "Storage account name" on this page.

> ***Refer to your IT Policies for Subscriptions and Resource Groups.***

***Figure A.1.2***

![](./media/image5.png)

For this setup we want to make Storage open to public, but you can change the setup to fit your organization needs. If Storage Account Network is configured with Private endpoints with Private virtual networks, make sure to add 2 subnets that Azure Databricks creates so Storage Account is accessible to Azure Databricks.

****Upon launching Azure Databricks, the service will create "Databricks-Private" and "Databricks-Public" subnets.****

***Figure A.1.3***

![](./media/image6.png)

Click "Next: Advanced" at the bottom of the page. On the Advanced tab, we want to be sure that Data Lake Storage Generation 2 _"Hierarchical namespace"_ is enabled.

***Figure A.1.4.***

![](./media/image7.png)

Configuration is complete. Press Review + create at the bottom of the page. Azure will validate what you have configured.

***Figure A.1.5.***

![](./media/image8.jpg)


Assuming your configuration passed the validation step, press “Create” button at the bottom of the page. It will take Azure a few minutes to create your storage account.


### **Part 2 - Create an Active Directory and perform "App Registration"**

Now we need to create an Azure "App Registration" with appropriate permissions that Qlik Replicate will use when writing data to ADLSv2 storage. From the Azure portal home page, click on "Azure Active Directory".

***Figure A.2.0.***

![](./media/image9.png)

Click on "App registrations" on the left side of the screen.

***Figure A.2.1.***

![](./media/image10.png)

Select "App registrations" from side menu and click "_+ New registration"._

***Figure A.2.2***

![](./media/image11.png)

Choose a name for your App and press "Register" at the bottom of the page. You do not need to enter anything for the "Redirect URI". It is optional and not required in our case.

***Figure A.2.3***

![](./media/image12.png)

Registration is almost immediate. Make note of the location of the "Application (client) ID" and "Directory (tenant) ID" fields at the top of the page. You will need this information later when configuring Replicate.

***Figure A.2.4***<br>
![](./media/image13.png)

Next, we need to grant this application some basic permissions. Click on _API permissions_ on the left side of the page.

***Figure A.2.5***<br>
![](./media/image14.png) 

Click "_+ Add permission"_ followed by _Azure Data Lake_ under "Microsoft APIs" on the right side of the screen.

***Figure A.2.6*** 
![](./media/image15.jpeg)

***Figure A.2.7***<br>
![](./media/image16.png)

From there, check "user_impersonation" under "Delegated permissions" and then press "Add permissions" button at the bottom of the page

***Figure A.2.8***
![](./media/image17.jpg)   

Now we need to create a "secret" (essentially a password) for this API. Click on "Certificates & secrets" on the left side of the screen and click on "+ New client secret".***Figure A.2.9*** ![](./media/image18.jpg)

Enter a description, choose an expiration, and press "_Add."_

***Figure A.2.10***

![](./media/image19.png)

**IMPORTANT NOTE**: Make Note of value of "Secret" you generated. You must save the value of the secret you created before you leave this page. You will not be able to retrieve it again later.

***Figure A.2.11***

![](./media/image20.jpg)

### **Part 3 - Finalizing Configuration of the ADLS-2 Storage Account**

Now that we have created the Azure Active Directory App Registration, we need to return to the ADLS-2 storage account and finalize the configuration, so it is ready for use by Qlik Replicate. Return to the Azure Portal home page and then drill in to get back to the storage account we created. (Refer to Figure A1.3)

First, we need to create a "Container" for us to use for the Qlik Replicate. Select "Containers" on the left side of the storage account screen.

***Figure A.3.1***

![](./media/image21.jpg)

Inside your Storage account "+ Container" at the top of the page.

***Figure A.3.2***

![](./media/image22.png)

Enter a name for the Container and press "OK" to save it.

***Figure A.3.3***

![](./media/image23.jpg)

As a final step, we need to grant access to the storage account to the "App" that we created / registered previously. Click on "Access control (IAM)" on the left side of the page.

***Figure A.3.4***

![](./media/image24.jpg)

Click on "Add a role assignment" on right.
# Table of Contents# Table of Contents
Once in the Screen on the right side of the screen select:

- **_Role:_** Storage Blob Data Contributor

- _Assign access to_: Azure AD user, group, or service principal

- _Select_: enter the name of the App you registered previously. (***Figure A.2.4***)

and press "Save" button at the bottom of the page.

***Figure A.3.5***

![](./media/image25.jpg)

### **Part 4 - Create a Target Directory in the File System**

To complete this part of the guide, you will need to create a target directory where Qlik Replicate will deliver data for the Databricks external tables.

To get started, go to your ADLS-2 storage account and click on Storage Explorer (preview). This will download Storage Explorer application on your computer. Select Storage Accounts and select Blob Container created

***Figure A.4.1***

![](./media/image26.png)

From there click on Blob Container Created and Add New Folders for Qlik Replicate (In this example we used "test" as folder name)

***Figure A.4.2***

![](./media/image27.jpg)

In the next section we will configure Azure Databricks to make use of this storage and make it ready to ingest data delivered by Qlik Replicate.
