
## Exercise 3: Path B — Migrate with Azure DMS from Azure Blob Storage

### Estimated Duration: 45 Minutes

When the Managed Instance link can't be used, **Azure Database Migration Service (DMS)** can migrate from backups in **Azure Blob Storage**. SQL Server 2016 backs up directly to Blob with `BACKUP ... TO URL`, and DMS restores the full and log backups to SQL MI — no SMB share or self-hosted integration runtime. In this exercise you migrate a second copy, **WideWorldImportersB<inject key="Suffix" enableCopy="false"/>**.

## Lab Objectives

- Task 1: Create a SAS token for the backup container
- Task 2: Back up the database to Azure Blob Storage
- Task 3: Create the Azure Database Migration Service
- Task 4: Create and run an online migration from Blob Storage
- Task 5: Back up the transaction log and cut over

## Task 1: Create a SAS token for the backup container

1. In the portal, open **Storage accounts** > **sqlmistore<inject key="Suffix" enableCopy="false"/>**.
1. Under **Data storage**, select **Containers** > **+ Container**, name it **migration**, and **Create**.
1. Select the **migration** container > **Settings** > **Shared access tokens**.
1. Set **Permissions** to Read, Add, Create, Write, Delete, List; set an **Expiry** one day out; select **Generate SAS token and URL**.
1. Copy the **Blob SAS token** into Notepad (drop the leading `?` when you use it).

## Task 2: Back up the database to Azure Blob Storage

1. On the **<inject key="SQLVM Name" enableCopy="false"/>** VM, in SSMS on the source connection, paste the following (replace **SUFFIX** with **<inject key="Suffix" />** and `<SAS-token>`), and **Execute**:

    ```sql
    USE master;
    GO

    CREATE CREDENTIAL [https://sqlmistoreSUFFIX.blob.core.windows.net/migration]
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
         SECRET = '<SAS-token>';
    GO

    BACKUP DATABASE WideWorldImporters
    TO URL = 'https://sqlmistoreSUFFIX.blob.core.windows.net/migration/wwi-b/WideWorldImporters_full.bak'
    WITH CHECKSUM, COMPRESSION, MAXTRANSFERSIZE = 4194304, BLOCKSIZE = 65536;
    GO
    ```

1. Confirm the backup succeeds and the **wwi-b** folder contains the file in the portal.

## Task 3: Create the Azure Database Migration Service

1. Portal > search **Azure Database Migration Services** > **+ Create**.
1. Source server type **SQL Server**, target **Azure SQL Managed Instance**, pricing **Database Migration Service**, then **Select**.
1. Resource group **<inject key="Resource Group Name" enableCopy="false"/>**, name **wwi-dms-<inject key="Suffix" enableCopy="false"/>**, same region, then **Review + create** > **Create** > **Go to resource**.

## Task 4: Create and run an online migration from Blob Storage

1. On **wwi-dms-<inject key="Suffix" enableCopy="false"/>**, select **+ New migration**.
1. Set source **SQL Server**, target **Azure SQL Managed Instance**, mode **Online migration**, backup location **Azure Blob Storage**, then **Select**.
1. On **Target**, select your subscription, **SQLMI-Shared-RG**, and the managed instance; **Next**.
1. On **Database and backup**, enter: source **WideWorldImporters**, target **WideWorldImportersB<inject key="Suffix" enableCopy="false"/>**, storage account **sqlmistore<inject key="Suffix" enableCopy="false"/>**, container **migration**, folder **wwi-b**; **Next**.
1. Review and **Start migration**.
1. On **Migrations**, **Refresh** until the full backup shows **Restored** and status is **Ready for cutover** (~10–15 min).

## Task 5: Back up the transaction log and cut over

1. On the source connection, run (replace **SUFFIX**):

   ```sql
   USE WideWorldImporters;
   GO
   INSERT [dbo].[Game] (Title, Description, Rating, IsOnlineMultiplayer)
   VALUES ('Ocean Odyssey', 'Dive into an underwater world full of hidden treasures.', 'E', 1);
   GO
   BACKUP LOG WideWorldImporters
   TO URL = 'https://sqlmistoreSUFFIX.blob.core.windows.net/migration/wwi-b/WideWorldImporters_log1.trn'
   WITH CHECKSUM, COMPRESSION, MAXTRANSFERSIZE = 4194304, BLOCKSIZE = 65536;
   GO
   ```

1. In the portal, **Refresh** until **WideWorldImporters_log1.trn** shows **Restored**.
1. Select **Complete cutover**, confirm **0** pending log backups, check the box, and **Complete cutover**.
1. **Refresh** until the status is **Succeeded**.
1. On the SQL MI connection, verify (replace **SUFFIX**):

   ```sql
   USE WideWorldImportersBSUFFIX;
   GO
   SELECT Title FROM dbo.Game WHERE Title IN ('Space Adventure', 'Ocean Odyssey');
   ```

## Summary

You backed up a SQL Server 2016 database directly to Azure Blob Storage and used Azure DMS to run an online migration to SQL MI, with no file share or self-hosted integration runtime.

### You have successfully completed the exercise. Now click on **Next >>** from the lower right corner to move on to the next exercise.
