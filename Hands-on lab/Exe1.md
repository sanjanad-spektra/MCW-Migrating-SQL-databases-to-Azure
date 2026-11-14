## Exercise 1: Assess the SQL Server 2016 database with Azure Arc

### Estimated Duration: 30 Minutes

## Lab Scenario

In this exercise, you connect to the on-premises SQL Server 2016 database, prepare it for migration, optionally capture a Query Store performance baseline, and assess its readiness for Azure SQL Managed Instance using **Azure Arc**. The Azure Arc migration assessment replaces the retired Data Migration Assistant. It runs from the Arc-enabled SQL Server resource in the Azure portal and gives you compatibility findings and a SKU recommendation.

## Lab Objectives

In this exercise, you will complete the following tasks:

- Task 1: Connect to the SQL Server 2016 VM and prepare the WideWorldImporters database
- Task 2: Capture a Query Store performance baseline (optional)
- Task 3: Review the Azure Arc-enabled SQL Server
- Task 4: Run the Azure Arc migration assessment

## Task 1: Connect to the SQL Server 2016 VM and prepare the database

In this task, you connect to the **<inject key="SQLVM Name" enableCopy="false"/>** VM, confirm the SQL Server build, and configure the `WideWorldImporters` database for migration.

1. On the **Azure portal**, in the **Search resources** box at the top, search for **Resource groups (1)** and then select **Resource groups (2)** from the results.

   ![](media/gs-g-et-4.png)

1. On the **Resource groups** page, select the resource group **<inject key="Resource Group Name" enableCopy="false"/>** from the list.

   ![](media/gs-g-et-5.png)

1. In the list of resources for your resource group, select the **sql2022-<inject key="Suffix" enableCopy="false"/>** Virtual Machine.

    ![](media/sql50-1.png)

1. On the **sql2022-<inject key="Suffix" enableCopy="false"/>** virtual machine **Overview** page, click **Connect (1)** and then select **Connect (2)** from the drop-down menu.

    ![](media/gs-g-et-6.png)

1. On the **sql2022-<inject key="Suffix" enableCopy="false"/> | Connect** page, click on **Download RDP file** under **Native RDP**. 
  
    ![](media/E1T1S6-1809.png)

1. Click on **Keep**, on the Downloads pop-up. 

    ![](media/E1T1S7-0701.png)

1. Click on **Open file** to open the RDP session.

    ![](media/E1T1S8-0701.png)

1. On the RDP pop-up, **check (1)** "I understand and want to allow RDP files to open on this device for my account" and then click **OK (2)**.

    ![](media/E1T1S8-0701-1.png)

1. Next, on RDP tab, ensure that the **Clipboard** option is enabled, and then select **Connect**.

   ![The Passed tab is highlighted, and VA1219 is entered into the search filter. VA1219 with a status of PASS is highlighted in the results.](media/datamod17.png "Passed")

1. Enter the following credentials when prompted, and then select **OK**:

   - **Username**: `sqlmiuser`
   - **Password**: `Password.1234567890`

      ![The credentials specified above are entered into the Enter your credentials dialog.](media/E1T1S10-1809.png "Enter your credentials")

1. On **The identity of the remote computer cannot be verified** pop-up window, click **Yes** to confirm and proceed with connecting to your virtual machine.

    ![](media/sql56.png)

1. On the VM, open **SQL Server Management Studio (SSMS)** from the Start menu.

1. In the **Connect to Server** dialog, enter `<inject key="SQLVM Name" enableCopy="false"/>` into the **Server name** box, ensure **Windows Authentication** is selected, check **Trust server certificate**, and then select **Connect**.

1. Expand **Databases** in Object Explorer and verify that the **WideWorldImporters** database is listed.

1. Select **New Query**, paste the following script, and then select **Execute**. The script confirms the SQL Server build, sets the recovery model to **FULL**, and enables **Service Broker**.

    ```sql
    -- Build 13.0.7xxx = SQL Server 2016 SP3 with the Azure Connect pack (required for the Managed Instance link)
    SELECT SERVERPROPERTY('ProductVersion') AS ProductVersion,
           SERVERPROPERTY('ProductLevel')   AS ProductLevel,
           SERVERPROPERTY('Edition')        AS Edition;
    GO

    USE master;
    GO

    ALTER DATABASE WideWorldImporters SET
    RECOVERY FULL,
    ENABLE_BROKER WITH ROLLBACK IMMEDIATE;
    GO
    ```

1. In the results, confirm that **ProductVersion** starts with **13.0.7** and **ProductLevel** is **SP3**.

   >**Note:** SQL Server 2016 SP3 alone is build 13.0.6300. The **Azure Connect pack for SQL Server 2016** raises the build to 13.0.7000 or later and adds the components the Managed Instance link needs.

## Task 2: Capture a Query Store performance baseline (optional)

In this task, you turn on **Query Store** on the source database and run a sample workload. Query Store data is stored inside the database, so it travels with the database to SQL MI. In Exercise 4, you run the same workload on SQL MI and compare the results.

1. In SSMS, select **New Query**, paste the following script, and then select **Execute** to enable Query Store:

    ```sql
    USE master;
    GO

    ALTER DATABASE WideWorldImporters SET QUERY_STORE = ON;
    ALTER DATABASE WideWorldImporters SET QUERY_STORE (
        OPERATION_MODE = READ_WRITE,
        QUERY_CAPTURE_MODE = ALL,
        INTERVAL_LENGTH_MINUTES = 5);
    GO
    ```

1. Select **New Query**, paste the following sample workload, and then select **Execute**. The `GO 50` batch separator runs each query 50 times.

    ```sql
    USE WideWorldImporters;
    GO

    -- Leaderboard query used by the web application
    SELECT TOP (50) GamerTag, Name, GamerScore, GamesPlayed
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

1. In Object Explorer, expand **WideWorldImporters** > **Query Store**, and then double-click **Top Resource Consuming Queries**. Note the **duration** and **CPU time** of the Leaderboard query — you compare these values after the migration.

## Task 3: Review the Azure Arc-enabled SQL Server

The lab environment has already connected the SQL Server 2016 VM to Azure through **Azure Arc**. In this task, you review the Arc-enabled SQL Server resource.

1. In the **Azure portal**, search for **Azure Arc** and select it.

1. Under **Data services**, select **SQL Server instances**, and then select the **<inject key="SQLVM Name" enableCopy="false"/>** instance.

1. On the **Overview** page, review the instance details: **Version** (SQL Server 2016), **Edition**, **License type**, and **Host**.

1. In the left menu, select **Databases** and confirm that **WideWorldImporters** is listed.

   >**Note:** Azure Arc gives you one place to manage SQL Server outside Azure — inventory, best-practices and migration assessments, and Extended Security Updates for out-of-support versions such as SQL Server 2016.

## Task 4: Run the Azure Arc migration assessment

In this task, you run a migration assessment and review the database's readiness for **Azure SQL Database** and **Azure SQL Managed Instance**.

1. On the **<inject key="SQLVM Name" enableCopy="false"/>** Arc-enabled SQL Server resource, under **Migration** in the left menu, select **Database migration**.

1. On the **Assessment** tab, select **Run assessment** if there are no results yet. The assessment takes about 5–10 minutes.

   >**Note:** If an assessment already ran automatically, its results appear immediately.

1. Review the **Azure SQL Database** readiness for **WideWorldImporters**. The assessment flags features that Azure SQL Database doesn't support, including **Service Broker**, so the database isn't ready for Azure SQL Database.

1. Review the **Azure SQL Managed Instance** readiness. The database is **Ready** for SQL MI with no blocking issues.

1. Review the **SKU recommendation** for SQL MI — service tier, hardware, vCores and storage — based on the performance data collected from the source server.

   >**Note:** The database, including the Service Broker feature, can be migrated as-is, giving WWI a fully managed PaaS database with _near 100% compatibility_, including the features that prevented them from using Azure SQL Database.

   > **Alternative:** If Azure Arc isn't available in your environment, you can assess the database with **Azure Migrate** instead (discovery and assessment using the Azure Migrate appliance).

## Summary

In this exercise, you prepared the `WideWorldImporters` database on SQL Server 2016 SP3, optionally captured a Query Store baseline, and used the Azure Arc migration assessment to confirm that Azure SQL Managed Instance is the right target.

### You have successfully completed the exercise. Now click on **Next >>** from the lower right corner to move on to the next exercise.
