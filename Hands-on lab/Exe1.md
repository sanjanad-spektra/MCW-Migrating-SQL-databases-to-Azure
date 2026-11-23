# Exercise 1: Assess the SQL Server 2016 database with Azure Arc

### Duration: 30 Minutes

## 📖 Overview

In this exercise, you will prepare an on-premises SQL Server 2016 database for migration and use Azure Arc to evaluate its readiness for Azure.

You will first connect to the SQL Server environment and prepare the **<inject key="Database Name" enableCopy="false"/>**. You will then review the SQL Server instance through Azure Arc and run a migration assessment to identify compatibility considerations and determine the most suitable Azure target.

The assessment will help you understand whether the database can be migrated to Azure SQL Managed Instance, SQL Server on Azure VM, or Azure SQL Database, and will provide a SKU recommendation for the selected target.

## 🎯 Objectives

In this exercise, you will complete the following tasks:

- Task 1: Connect to the SQL Server 2016 VM and prepare the WideWorldImporters database
- Task 2: Capture a Query Store performance baseline (optional)
- Task 3: Review the Azure Arc-enabled SQL Server
- Task 4: Run the Azure Arc migration assessment

## Task 1: Connect to the SQL Server 2016 VM and prepare the database

In this task, you will connect to the **<inject key="SQLVM Name" enableCopy="false"/>** VM, verify the SQL Server configuration, and prepare the **<inject key="Database Name" enableCopy="false"/>** database for migration.

1. On the lab VM desktop, double-click the **Azure Portal** shortcut.

    ![](media/azureportal-1.png)

1. On the **Azure portal**, in the **Search resources** box at the top, search for **Resource groups (1)** and then select **Resource groups (2)** from the results.

   ![](media/gs-g-et-4.png)

1. On the **Resource groups** page, select the resource group **<inject key="Resource Group Name" enableCopy="false"/>** from the list.

    ![](media/MG-image1.png)

1. In the list of resources for your resource group, select the **<inject key="Sqlvm Name" enableCopy="false"/>** Virtual Machine.

    ![](media/MG-image2.png)

1. On the **<inject key="Sqlvm Name" enableCopy="false"/>** virtual machine, on the **Overview** page, select **Connect (1)**, and then select **Connect (2)** from the drop-down menu.

    ![](media/MG-image3.png)

1. On the **<inject key="Sqlvm Name" enableCopy="false"/>| Connect** page, click on **Download RDP file** under **Native RDP**. 
  
    ![](media/MG-image4.png)

1. Click on **Keep**, on the Downloads pop-up. 

1. Click on **Open file** to open the RDP session.

1. On the RDP pop-up, **check (1)** "I understand and want to allow RDP files to open on this device for my account" and then click **OK (2)**.

    ![](media/E1T1S8-0701-1.png)

1. Next, in the **Remote Desktop Connection** window, ensure that **Clipboard (1)** is enabled, and then select **Connect (2)**.

   ![](media/datamod17.png)

1. Enter the following credentials when prompted, and then select **OK**:

   - **Username**: `sqlmiuser`
   - **Password**: `Password.1234567890`

      ![](media/E1T1S10-1809.png)

1. On **The identity of the remote computer cannot be verified** pop-up window, click **Yes** to confirm and proceed with connecting to your virtual machine.

    ![](media/MG-image129.png)

1. On the VM, open **SQL Server Management Studio (SSMS)** from the Start menu.

    ![](media/MG-image5.png)
   
1. In the **Connect to Server** dialog, enter `sqlvm2016-sp3` **(1)** into the **Server name** box, ensure **Windows Authentication (2)** is selected, check **Trust server certificate (3)**, and then select **Connect (4)**.

   ![](media/MG-image131.png)
   
1. Expand **Databases (1)** in **Object Explorer** and verify that the **<inject key="Database Name" enableCopy="false"/>(2)** database is listed.

    ![](media/MG-image7.png)

1. In SQL Server Management Studio (SSMS), right-click the **SQL Server instance (1)** in Object Explorer and select **New Query (2)**.

   ![](media/MG-image8.png)
   
1. Paste the following script and (replace SUFFIX with <inject key="Suffix" enableCopy="false"/>) and then select **Execute**. The script confirms the SQL Server build, sets the recovery model to **FULL**, and enables **Service Broker**.

    ```sql
    -- Build 13.0.7xxx = SQL Server 2016 SP3 + Azure Connect pack; Edition must be Developer/Standard/Enterprise
    SELECT SERVERPROPERTY('ProductVersion') AS ProductVersion,
           SERVERPROPERTY('ProductLevel')   AS ProductLevel,
           SERVERPROPERTY('Edition')        AS Edition;
    GO
    
    USE master;
    GO
    
    ALTER DATABASE WideWorldImportersSUFFIX SET
    RECOVERY FULL,
    ENABLE_BROKER WITH ROLLBACK IMMEDIATE;
    GO
    ```

    ![](media/MG-image132.png)
   
1. Confirm Edition = Developer (or Standard/Enterprise), ProductVersion starts with 13.0.7, ProductLevel = SP3.

   ![](media/MG-image130.png)
   
     >**Note:**  SP3 alone is 13.0.6300; the Azure Connect pack raises it to 13.0.7000+. The MI link requires Developer/Standard/Enterprise - Evaluation is not supported. If it shows Evaluation, upgrade via Installation Center > Maintenance > Edition Upgrade.

## Task 2: Capture a Query Store performance baseline (optional)

In this optional task, you will enable **Query Store** and capture a performance baseline to compare query performance before and after migration.

1. In SSMS, select **New Query**, paste the following script and(replace SUFFIX with <inject key="Suffix" enableCopy="false"/>) then select **Execute** to enable Query Store:

    ```sql
    USE master;
    GO
    ALTER DATABASE WideWorldImportersSUFFIX SET QUERY_STORE = ON;
    ALTER DATABASE WideWorldImportersSUFFIX SET QUERY_STORE (
        OPERATION_MODE = READ_WRITE, QUERY_CAPTURE_MODE = ALL, INTERVAL_LENGTH_MINUTES = 5);
    GO
    ```
   ![](media/MG-image33.png)
    
1. Select **New Query**, paste the following sample workload, and (replace SUFFIX with <inject key="Suffix" enableCopy="false"/>) then select **Execute**. The `GO 50` batch separator runs each query 50 times.

    ```sql
    USE WideWorldImportersSUFFIX;
    GO
    -- Leaderboard query
    SELECT TOP (50) GamerId, GamerScore, GamesPlayed
    FROM dbo.Leaderboard
    ORDER BY GamerScore DESC;
    GO 50
    -- Catalog query
    SELECT Title, Rating
    FROM dbo.Game
    WHERE IsOnlineMultiplayer = 1
    ORDER BY Title;
    GO 50
    ```

    ![](media/MG-image34.png)
   
1. In Object Explorer, expand **<inject key="Database Name" enableCopy="false"/>** > **Query Store**, and then double-click **Top Resource Consuming Queries**. Note the **duration** and **CPU time** of the Leaderboard query - you compare these values after the migration.

   ![](media/MG-image35.png)

### Task 3: Review the Azure Arc-enabled SQL Server

In this task, you will review the SQL Server instance connected through **Azure Arc** and verify that the database is available for assessment.

1. In the **Azure portal**, search for **Azure Arc** and select it.

    ![](media/MG-image12.png)

1. Under **Infrastructure**, select **Machines**, then verify the Arc agent status and click **sqlvm2016-sp3**.

   ![](media/MG-image143.png)

1. On the **Azure Arc** page, expand **Data services (1)** and select **SQL Server instances (2)**. On the **SQL Server instances** page, select your **<inject key="Resource Group Name" enableCopy="false"/>** in the **Resource group** filter, and then select **sqlvm2016-sp3 (4)**.

     ![](media/MG-image145.png)
   
1. On the **Overview** page, review the instance details: version (SQL Server 2016), edition, license type, and host.

    ![](media/MG-image14.png)
   
1. In the left menu, select **Databases** and confirm that **<inject key="Database Name" enableCopy="false"/></inject>** is listed.

    ![](media/MG-image15.png)
   
   >**Note:** Azure Arc gives you one place to manage SQL Server outside Azure - inventory, best-practices and migration assessments, and Extended Security Updates for out-of-support versions such as SQL Server 2016.

## Task 4: Run the Azure Arc migration assessment

In this task, you will run an **Azure Arc migration assessment**, review compatibility results, and confirm the recommended migration target and SKU.

1. On the **sqlvm2016-sp3** SQL Server resource page, in the left menu, under **Migration**, select **Assessments**. From the toolbar, select **Run assessment**. The assessment may take a few minutes to complete.

     ![](media/MG-image210.png) 

   >**Note:** Azure Arc also runs assessments automatically on a weekly schedule. The **Run assessment** button runs one on demand, which is what this task needs. If the page shows "No assessment run yet," select **Run assessment** and wait.

1. When the assessment completes, the **Assessment results** open with a **Migration strategy** of **Modernize to PaaS** and an **Overview** showing three target options side by side:

   - **Azure SQL MI** - **Ready** (0 migration issues)
   - **SQL Server on Azure VM** - Ready
   - **Azure SQL DB** - **Not ready**

      ![](media/MG-image211.png) 
      
1. Under the **Azure SQL DB** card, select **View assessment details**. In **Migration target type**, ensure **Azure SQL DB** is selected, and open the **Compatibility** tab. Under **Database compatibility**, note the blocking finding: **"Service Broker feature is not supported in Azure SQL Database"** (Issue). This is why Azure SQL Database is ruled out.

   ![](media/MG-image214.png)
    
   ![](media/MG-image212.png) 
   
1. Change **Migration target type** to **Azure SQL Managed Instance**.

   ![](media/MG-image20.png)
   
1. Confirm it shows **Ready** with no blocking issues. Then review the **SKU Recommendation** tab for the recommended service tier, vCores and storage, based on the source server's performance data.

    ![](media/MG-image213.png) 

   >**Note:** The database, including the Service Broker feature, can be migrated as-is, giving WWI a fully managed PaaS database with _near 100% compatibility_, including the features that prevented them from using Azure SQL Database.

   > **Alternative:** If Azure Arc isn't available in your environment, you can assess the database with **Azure Migrate** instead (discovery and assessment using the Azure Migrate appliance).

### 🧾 Summary

In this exercise, you prepared the WideWorldImporters database, reviewed its Azure Arc connection, and assessed its migration readiness. The assessment identified Azure SQL Managed Instance as the suitable target and provided a recommended SKU based on the workload.

#### You have successfully completed the exercise. Select Next >> to continue.
