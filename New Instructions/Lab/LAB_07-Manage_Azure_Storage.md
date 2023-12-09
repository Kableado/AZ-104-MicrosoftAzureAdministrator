---
lab:
    title: 'Lab 07: Manage Azure storage'
    module: 'Administer Azure Storage'
---

# Lab 07 - Manage Azure Storage
# Student lab manual

## Lab scenario

Your organization is currently storing data in on-premises data stores. The majority of these files are not accessed frequently. You would like to minimize the cost of storage by placing infrequently accessed files in lower-priced storage tiers. You also plan to explore different protection mechanisms that Azure Storage offers, including network access, authentication, authorization, and replication. Finally, you want to determine to what extent Azure Files is suitable for hosting your on-premises file shares.

**Note:** An **[interactive lab simulation](https://mslabs.cloudguides.com/guides/AZ-104%20Exam%20Guide%20-%20Microsoft%20Azure%20Administrator%20Exercise%2011)** is available that allows you to click through this lab at your own pace. You may find slight differences between the interactive simulation and the hosted lab, but the core concepts and ideas being demonstrated are the same. 

## Objectives

In this lab, you will:

+ Task 1: Provision the lab environment
+ Task 2: Create and configure Azure Storage accounts
+ Task 3: Manage blob storage
+ Task 4: Manage authentication and authorization for Azure Storage
+ Task 5: Create and configure an Azure Files shares
+ Task 6: Manage network access for Azure Storage

## Estimated timing: 30 minutes

## Architecture diagram

![image](./media/az104-lab07-architecture-diagram.png)


### Instructions

## Exercise 1

## Task 1: Provision the lab environment

In this task, you will deploy an Azure virtual machine by using an existing template. This VM will be used later in the lab to work with Azure storage.

1. If necessary, download the **\\Allfiles\\Labs\\07\\az104-vm1-template.json** and **\\Allfiles\\Labs\\07\\az104-vm1-parameters.json** lab files to your computer.

1. Sign in to the [Azure portal](https://portal.azure.com). 

1. From the Azure portal, search for and select `Deploy a custom template`.

1. On the custom deployment page, select **Build you own template in the editor**.

1. On the edit template page, select **Load file**.

1. Locate and select the **\\Allfiles\\Labs\\07\\az104-vm1-template.json** file and select **Open**.

    ![image](./media/az104-lab07-template1.png)

1. Select **Save**.

1. On the custom deployment page, select **Edit parameters**.

1. On the edit parameters page, select **Load file**. Locate and select the **\\Allfiles\\Labs\\07\\az104-vm1-parameters.json** file and select **Open**.

    ![image](./media/az104-lab07-parameters1.png)

1. Select **Save**.

1. Use the following information to complete the fields on the custom deployment page.

    | Setting       | Value         | 
    | ---           | ---           |
    | Subscription  | Your Azure subscription |
    | Resource group| `az104-rg1` (If necessary, select **Create new**)
    | Region        | **East US**   |
    | VM Size       | **Standard DS2 v3** |
    | Admin Username| `Student` |
    | Password      | Provide a secure password |

    ![image](./media/az104-lab07-deploy1.png)

1. Select **Review + Create**, and then select **Create**.

    >**Note**: Do not wait for the deployments to complete, but proceed to the next task.

    >**Note**: If you got an error stating the VM size is not available, perform the steps again, select **Change size** and try a SKU that is available in your subscription.


## Task 2: Create and configure Azure Storage accounts

In this task, you will create and configure an Azure Storage account. Azure storage accounts can be used for blob, file, queue, and table storage. You will use this storage account with the virtual machine that you created in the previous task.

1. In the Azure portal, search for and select **Storage accounts**, and then click **+ Create**.

1. On the **Basics** tab of the **Create storage account** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription          | the name of your Azure subscription  |
    | Resource group        | **az104-rg1** |
    | Storage account name  | any globally unique name between 3 and 24 in length consisting of letters and digits |
    | Region                | **East US**  |
    | Performance           | **Standard** |
    | Redundancy            | **Locally redundant storage (LRS)** |

    ![image](./media/az104-lab07-storage1.png)

1. Click **Next: Advanced >**. 

1. On the **Advanced** tab, review the available options, accept the defaults, and click **Next: Networking >**.

1. On the **Networking** tab, review the available options, accept the default option **Enable public access from all networks** and click **Next: Data protection >**.

1. On the **Review** tab, review the available options, accept the defaults, click **Review + Create**, wait for the validation process to complete and click **Create**.

    >**Note**: Wait for the Storage account to be created. This should take about 2 minutes.

    ![image](./media/az104-lab07-storage-review.png)

1. On the deployment blade, click **Go to resource** to display the Azure Storage account blade.

## Task 3: Manage blob storage

In this task, you will create a blob container and upload a blob into it. Blob containers are directory-like structures that store blobs (unstructured data) in the storage account. You will interact with the blob container and blob object that you create later in the lab.

1. On the Storage account blade, in the **Data storage** section, click **Containers**.

1. Click **+ Container** and create a container with the following settings:

    | Setting | Value |
    | --- | --- |
    | Name | `data`  |
    | Public access level | **Private (no anonymous access)** |

    ![image](./media/az104-lab07-create-container.png)

1. In the list of containers, click **data** and then click **Upload**.

1. Browse to **\\Allfiles\\Labs\\07\\LICENSE** on your lab computer and click **Open**.

1. On the **Upload blob** blade, expand the **Advanced** section and specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Blob type | **Block blob** |
    | Block size | **4 MB** |
    | Access tier | **Hot** |
    | Upload to folder | **licenses** |

    > **Note**: Access tier can be set for individual blobs.

    ![image](./media/az104-lab07-upload-blob.png)

1. Click **Upload**.

    > **Note**: Note that the upload automatically created a subfolder named **licenses**.

1. Back on the **data** blade, click **licenses** and then click **LICENSE**.

1. On the **licenses/LICENSE** blade, review the available options.

    > **Note**: You have the option to download the blob, change its access tier (it is currently set to **Hot**), acquire a lease, which would change its lease status to **Locked** (it is currently set to **Unlocked**) and protect the blob from being modified or deleted, as well as assign custom metadata (by specifying an arbitrary key and value pairs). You also have the ability to **Edit** the file directly within the Azure portal interface, without downloading it first. You can also create snapshots, as well as generate a SAS token (you will explore this option in the next task).

## Task 4: Manage authentication and authorization for Azure Storage

In this task, you will configure authentication and authorization for Azure Storage. By default, new Azure storage accounts do not allow you to set containers to anonymous access. You can choose to override this for the storage account if you need to be able to allow anonymous access, or you can use other authentication options to access blobs.

1. On the **licenses/LICENSE** blade, on the **Overview** tab, click **Copy to clipboard** button next to the **URL** entry.

1. Open another browser window by using InPrivate mode and navigate to the URL you copied in the previous step.

1. You should be presented with an XML-formatted message stating **ResourceNotFound** or **PublicAccessNotPermitted**.

    > **Note**: This is expected, since the container you created has the public access level set to **Private (no anonymous access)**.

1. Close the InPrivate mode browser window, return to the browser window showing the **licenses/LICENSE** blade of the Azure Storage container, and switch to the the **Generate SAS** tab.

1. On the **Generate SAS** tab of the **licenses/LICENSE** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Signing key | **Key 1** |
    | Permissions | **Read** |
    | Start date | yesterday's date |
    | Start time | current time |
    | Expiry date | tomorrow's date |
    | Expiry time | current time |
    | Allowed IP addresses | leave blank |
    
    ![image](./media/az104-lab07-sas1.png)

1. Click **Generate SAS token and URL**.

1. Click **Copy to clipboard** button next to the **Blob SAS URL** entry.

1. Open another browser window by using InPrivate mode and navigate to the URL you copied in the previous step.

    > **Note**: You should be able to view the content of the file by downloading it and opening it with Notepad. If you receive a Windows SmartScreen error, continue to the page.

    > **Note**: Save the blob SAS URL. You will need it later in this lab.

1. Close the InPrivate mode browser window, return to the browser window showing the **licenses/LICENSE** blade of the Azure Storage container, and from there, navigate back to the **data** blade.

1. Click the **Switch to the Azure AD User Account** link next to the **Authentication method** label.

    > **Note**: You can see an error when you change the authentication method (the error is *"You do not have permissions to list the data using your user account with Microsoft Entra ID"*). It is expected.  

    > **Note**: At this point, you do not have permissions to change the Authentication method.

    ![image](./media/az104-lab07-storage-error.png)

1. On the **data** blade, click **Access Control (IAM)**.

1. On the **Check access** tab, click **Add role assignment**.

1. On the **Add role assignment** blade, specify the following settings:

    | Setting | Value |
    | --- | --- |
    | Role | **Storage Blob Data Owner** |
    | Assign access to | **User, group, or service principal** |
    | Members | the name of your user account |

1. Click **Review + Assign** and then **Review + assign**, and return to the **Overview** blade of the **data** container and verify that you can change the Authentication method to (Switch to Azure AD User Account).

    > **Note**: It might take about 5 minutes for the change to take effect.

## Task 5: Create and configure an Azure Files shares

In this task, you will create and configure Azure Files shares. Azure File Shares allow you to interact with Azure storage using either SMB or NFS protocols. You will then map a network drive from the VM that you deployed to the file share that you create.

> **Note**: Before you start this task, verify that the virtual machine you provisioned in the first task of this lab is running.

1. In the Azure portal, navigate back to the blade of the storage account you created in the first task of this lab and, in the **Data storage** section, click **File shares**.

1. Click **+ File share** and on the **Basics** tab give the file share a name, `share1`. Review the other settings on this tab. 

1. Move to the **Backup** tab, and ensure **Enable Backup** is **not** checked.

1. Click **Review and create**, and then **Create**. Wait for the file share to deploy.

    ![image](./media/az104-lab07-create-share.png)

1. Click the newly created file share and note the information available on the **share1** blade.

1. Click **Browse** and note that there are no files or folders in the new file share. Click **Connect**.

1. On the **Connect** blade, ensure that the **Windows** tab is selected. Below you will find a button with the label **Show Script**. Click on the button and you will find grey textbox with a script, in the bottom right corner of that box hover over the pages icon and click **Copy to clipboard**.

1. In the Azure portal, search for and select **Virtual machines**, and, in the list of virtual machines, click **az104-vm1**.

1. On the **az104-vm1** blade, in the **Operations** section, click **Run command**.

1. On the **az104-vm1** - Run command** blade, click **RunPowerShellScript**.

1. On the **Run Command Script** blade, paste the script you copied earlier in this task into the **PowerShell Script** pane and click **Run**.

    ![image](./media/az104-lab07-run-command.png)

1. Verify that the script completed successfully.

1. Replace the content of the **PowerShell Script** pane with the following script and click **Run**:

   ```powershell
   New-Item -Type Directory -Path 'Z:\folder1'

   New-Item -Type File -Path 'Z:\folder1\file1.txt'
   ```

1. Verify that the script completed successfully.

1. Navigate back to the **share1 \| Browse** file share blade, click **Refresh**, and verify that the **folder1** appears in the list of folders.

1. Click **folder1** and verify that **file1.txt** appears in the list of files.

    ![image](./media/az104-lab07-file-browse.png)

## Task 6: Manage network access for Azure Storage

In this task, you will configure network access for Azure Storage. Earlier in this lab when you created the storage account, it was configured to allow connections from any public IP address. In this lab, you will restrict the network access for the storage account to your specific IP address.

1. In the Azure portal, navigate back to the blade of the storage account you created in the first task of this lab and, in the **Security + Networking** section, click **Networking**.

1. From the **Firewalls and virtual networks** tab, click the **Enabled from selected virtual networks and IP addresses** option and review the configuration settings that become available once this option is enabled.

    > **Note**: You can use these settings to configure direct connectivity between Azure virtual machines on designated subnets of virtual networks and the storage account by using service endpoints.

1. Click the checkbox **Add your client IP address** and select **Save**.

    ![image](./media/az104-lab07-storage-networking.png)

1. Open another browser window by using InPrivate mode and navigate to the blob SAS URL you generated in the previous task.

    > **Note**: If you did not record the SAS URL from task 4, you should generate a new one with the same configuration. Use Task 4 steps 4-6 as a guide for generating a new blob SAS URL. 

1. You should be presented with the content of **The MIT License (MIT)** page.

    > **Note**: This is expected, since you are connecting from your client IP address.

1. Close the InPrivate mode browser window, return to the browser window showing the **Networking** blade of the Azure Storage account.

## Review

Congratulations! You have successfully provisioned an Azure virtual machine and storage account, then configured the storage account for blob and file storage.