# Module 02 - Register & Scan

[< Previous Module](../modules/module01.md) - **[Home](../README.md)** - [Next Module>](../modules/module03.md)

## :thinking: Prerequisites

* An [Azure account](https://azure.microsoft.com/en-us/free/) with an active subscription.
* An Azure Azure Purview account (see [module 01](../modules/module01.md)).

## :hammer: Tools

* [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) (Download and Install)

## :loudspeaker: Introduction

To populate Azure Purview with assets for data discovery and understanding, we must register sources that exist across our data estate so that we can leverage the out of the box scanning capabilities. Scanning enables Azure Purview to extract technical metadata such as the fully qualified name, schema, data types, and apply classifications by parsing a sample of the underlying data. In this module, we will walk through how to register and scan data sources.

## :dart: Objectives

* Register and scan an Azure Data Lake Storage Gen2 account using the Azure Purview managed identity.
* Register and scan an Azure SQL Database using SQL authentication credentials stored in Azure Key Vault.

## Table of Contents

1. [Create an Azure Data Lake Storage Gen2 Account](#1-create-an-azure-data-lake-storage-gen2-account)
1. [Grant the Azure Purview Managed Identity Access](#2-grant-the-azure-purview-managed-identity-access)
1. [Upload Data to Azure Data Lake Storage Gen2 Account](#3-upload-data-to-azure-data-lake-storage-gen2-account)
1. [Create a Collection](#4-create-a-collection)
1. [Register a Source (ADLS Gen2)](#5-register-a-source-adls-gen2)
1. [Scan a Source with the Azure Purview Managed Identity](#6-scan-a-source-with-the-azure-purview-managed-identity)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 1. Create an Azure Data Lake Storage Gen2 Account

1. Sign in to the [Azure portal](https://portal.azure.com) with your Azure account and from the **Home** screen, click **Create a resource**.

    ![Azure Purview](../images/module01/01.01-create-resource.png)  

2. Under "Popular", click **Storage account**.

    ![Azure Purview](../images/module02/02.01-create-storage.png)

3. Provide the necessary inputs on the **Basics** tab.  
    > Note: The table below provides example values for Subscription, Resource Group, Purview account name, and Location for illustrative purposes only, ensure to specify values that make sense for your deployment.

    | Parameter  | Example Value |
    | --- | --- |
    | Subscription | `Azure Internal Access` |
    | Resource group | `purviewlab` |
    | Storage account name | `storage69426` |
    | Location | `(South America) Brazil South` |
    | Performance | `Standard` |
    | Account kind | `StorageV2 (general purpose v2)` |
    | Replication | `Locally-redundant storage (LRS)` |

    ![Azure Purview](../images/module02/02.02-storage-basics.png)

4. On the **Advanced** tab, set the **Hierarchal namespace** to **Enabled**.

    ![Azure Purview](../images/module02/02.03-storage-adls.png)

5. On the **Review + Create** tab, once the message in the ribbon returns "Validation passed", verify your selections and click **Create**.

    ![Azure Purview](../images/module02/02.04-storage-validate.png)

6. Wait several minutes while your deployment is in progress. Once complete, click **Go to resource**.

    ![Azure Purview](../images/module02/02.05-storage-goto.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 2. Grant the Azure Purview Managed Identity Access

To scan a source, Azure Purview requires a set of credentials with the necessary permissions. For Azure Data Lake Storage Gen2, Azure Purview supports the following authentication methods.

* Managed Identity (recommended)
* Service Principal
* Account Key

In this module we will walk through how to grant the Azure Purview Managed Identity the necessary access to successfully configure and run a scan.

1. From your Azure Data Lake Storage Gen2 account, select **Access Control (IAM)** from the left navigation menu.

    ![Azure Purview](../images/module02/02.06-storage-access.png)

2. Click **Add role assignments**.

    ![Azure Purview](../images/module02/02.07-storage-addrole.png)

3. Populate the role assignment prompt as per the table below, select the Azure Purview managed from the list, click **Save**.

    | Property  | Value |
    | --- | --- |
    | Role | `Storage Blob Data Reader` |
    | Assign access to | `User, group, or service principal` |
    | Select | `<purview-account-name>` |

    ![Azure Purview](../images/module02/02.08-storage-assignment.png)

4. Navigate to the **Role assignments** tab and confirm the Azure Purview managed identity has been assigned the Storage Blob Data Reader role. Tip: Filter **Scope** to `This resource` to limit the results.

    ![Azure Purview](../images/module02/02.09-storage-reader.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 3. Upload Data to Azure Data Lake Storage Gen2 Account

Before proceeding with the following steps, you will need to:

* Download and install [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/).
* Open Azure Storage Explorer.
* Sign in to Azure via **View > Account Management > Add an account...**.

1. Download a copy of the **[Bing Coronavirus Query Set](https://github.com/tayganr/purviewlab/raw/main/assets/BingCoronavirusQuerySet.zip)** to your local machine. Note: This data set was originally sourced from [Microsoft Research Open Data](https://msropendata.com/datasets/c5031874-835c-48ed-8b6d-31de2dad0654).

2. Locate the downloaded zip file via File Explorer and unzip the contents by right-clicking the file and selecting **Extract All...**.

    ![Extract zip file](../images/module02/02.10-explorer-unzip.png)

3. Click **Extract**.

    ![Extract](../images/module02/02.11-explorer-extract.png)

4. Open Azure Storage Explorer, click on the Toggle Explorer icon, expand the Azure Subscription to find your Azure Storage Account. Right-click on Blob Containers and select **Create Blob Container**. Name the container **raw**.

    ![Create Blob Container](../images/module02/02.12-explorer-container.png)

5. With the container name selected, click on the **Upload** button and select **Upload Folder...**.

    ![Upload Folder](../images/module02/02.13-explorer-upload.png)

6. Click on the **ellipsis** to select a folder.

    ![Browse](../images/module02/02.14-explorer-browse.png)

7. Navigate to the extracted **BingCoronavirusQuerySet** folder (e.g. Downloads\BingCoronavirusQuerySet) and click **Select Folder**.

    ![Folder](../images/module02/02.15-explorer-folder.png)

8. Click **Upload**.

    ![Upload](../images/module02/02.16-explorer-data.png)

9. Monitor the **Activities** until the transfer is complete.

    ![Transfer Complete](../images/module02/02.17-explorer-transfer.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 4. Create a Collection

1. Open Purview Studio, navigate to **Sources** and click **New collection**.

    ![New Collection](../images/module02/02.18-sources-collection.png)

2. Provide the collection a **Name** (e.g. Contoso) and click **Create**.

    > :bulb: **Did you know?**
    >
    > **Collections** can be used to logically group data sources. Collections can also belong to other Collections to form a hierarchy (e.g. Organization Name > Business Unit). Collections can collapse or expand by clicking the circle attached to the arrow between levels.

    ![Contoso Collection](../images/module02/02.19-sources-contoso.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 5. Register a Source (ADLS Gen2)

1. Open Purview Studio, navigate to **Sources** and click **Register**.

    ![Register](../images/module02/02.20-sources-register.png)

2. Select **Azure Data Lake Storage Gen2** and click **Continue**.

    ![Sources](../images/module02/02.21-sources-adls.png)

3. Select the **Azure subscription**, **Storage account name**, and **Collection**. Click **Register**.

    > :bulb: **Did you know?**
    >
    > At this point, we have simply registered a data source. Assets are not written to the catalog until after a scan has finished running.

    ![Source Properties](../images/module02/02.22-sources-properties.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## 6. Scan a Source with the Azure Purview Managed Identity

1. Open Purview Studio, navigate to **Sources**, and within the Azure Data Lake Storage Gen2 source tile, click the **New Scan** button.

    ![New Scan](../images/module02/02.23-scan-new.png)

2. Click **Test connection** to ensure the Azure Purview managed identity has the appropriate level of access to read the Azure Data Lake Storage Gen2 account. If successful, click **Continue**.

    ![Test Connection](../images/module02/02.24-scan-test.png)

3. Expand the hierarchy to see which assets will be within the scans scope, and click **Continue**.

    ![Scan Scope](../images/module02/02.25-scan-scope.png)

4. Select the system default scan rule set and click **Continue**.

    > :bulb: **Did you know?**
    >
    > **Scan Rule Sets** determine which **File Types** and **Classification Rules** are in scope. If you want to include a custom file type or custom classification rule as part of a scan, a custom scan rule set will need to be created.

    ![Scan rule set](../images/module02/02.26-scan-ruleset.png)

5. Select **Once** and click **Continue**.

    ![Scan Trigger](../images/module02/02.27-scan-trigger.png)

6. Click **Save and Run**.

    ![Run Scan](../images/module02/02.28-scan-run.png)

7. To monitor the progress of the scan run, click **View Details**.

    ![View Details](../images/module02/02.29-sources-details.png)

8. Click **Refresh** to periodically update the status of the scan. Note: It will take approximately 5 minutes to complete.

    ![Monitor Scan](../images/module02/02.30-sources-refresh.png)

<div align="right"><a href="#module-02---register--scan">↥ back to top</a></div>

## :tada: Summary

This module provided an overview of how to create a collection, register a source, and trigger a scan.
