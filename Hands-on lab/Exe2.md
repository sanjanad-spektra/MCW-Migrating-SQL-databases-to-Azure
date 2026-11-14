## Exercise 2: Path A — Migrate with Azure Arc and the Managed Instance link

### Estimated Duration: 60 Minutes

In this exercise, you migrate the `WideWorldImporters` database to Azure SQL Managed Instance using the **Managed Instance link**, starting from the Azure Arc-enabled SQL Server. The link uses distributed availability groups to replicate the database to SQL MI in near real time; cutover takes only seconds. For a SQL Server 2016 source, the link is one-way — once you cut over to SQL MI, you cannot fail back.

## Lab Objectives

- Task 1: Retrieve the SQL MI connection information
- Task 2: Check the Managed Instance link prerequisites
- Task 3: Start the migration with the Managed Instance link from Azure Arc
- Task 4: Verify near real-time replication
- Task 5: Cut over to SQL MI

## Task 1: Retrieve the SQL MI connection information

1. In the Azure portal, select the **Cloud Shell** icon, and choose **PowerShell**. On the **Getting started** pane, select **No storage account required**, select your subscription, and select **Apply**.

1. Run the following to get the SQL MI details:

   ```PowerShell
   $resourceGroup = "SQLMI-Shared-RG"
   az sql mi list --resource-group $resourceGroup --query "[].{name:name, fqdn:fullyQualifiedDomainName, tier:sku.tier}" -o table
   ```

1. Copy the **fqdn** value into Notepad.

## Task 2: Check the Managed Instance link prerequisites

The link requires SQL Server 2016 SP3 + Azure Connect pack (Exercise 1), the **Always On availability groups** feature enabled, a **database master key**, FULL recovery with a full backup, and network connectivity to SQL MI (TCP 5022 both directions, 11000–11999 from SQL Server to SQL MI). The environment has enabled the feature and opened the ports.

1. On the **<inject key="SQLVM Name" enableCopy="false"/>** VM, in SSMS, select **New Query**, paste the following, and **Execute**:

    ```sql
    SELECT SERVERPROPERTY('IsHadrEnabled') AS IsHadrEnabled;  -- want 1
    GO

    USE master;
    GO

    IF NOT EXISTS (SELECT 1 FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
        CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MiLink-Pa55w.rd-2026!';
    GO

    BACKUP DATABASE WideWorldImporters
    TO DISK = 'WideWorldImporters_full.bak'
    WITH CHECKSUM, INIT, COMPRESSION;
    GO
    ```

1. Confirm **IsHadrEnabled** is **1** and the backup succeeds.

   >**Note:** If IsHadrEnabled is 0, enable **AlwaysOn Availability Groups** in SQL Server Configuration Manager (SQL Server service > Properties > AlwaysOn High Availability), then restart the SQL Server service.

1. On the VM, in **Windows PowerShell**, run (replace `<sqlmi-fqdn>`):

    ```PowerShell
    Test-NetConnection -ComputerName <sqlmi-fqdn> -Port 5022
    ```

1. Confirm **TcpTestSucceeded** is **True**.

## Task 3: Start the migration with the Managed Instance link from Azure Arc

1. In the portal, go to **Azure Arc** > **SQL Server instances** > **<inject key="SQLVM Name" enableCopy="false"/>**.

1. Under **Migration**, select **Database migration**, then the **Migrate** tab.

1. Select **WideWorldImporters**, then **Migrate to Azure SQL Managed Instance**.

1. On **Target**, select your subscription, the **SQLMI-Shared-RG** resource group, and the managed instance. Select **Next**.

1. On **Migration method**, select **Online migration using Managed Instance link**, then **Next**.

1. On **Settings**, set **Target database name** to **WideWorldImporters<inject key="Suffix" enableCopy="false"/>**, then **Next**.

1. Review the summary and select **Start migration**. Seeding takes about 10–15 minutes.

1. Select **Refresh** until the status is **Replicating** (or **Ready for cutover**).

   > **Alternative:** You can create the same link from SSMS — right-click **WideWorldImporters** > **Azure SQL Managed Instance link** > **New...**. Use this if the Arc migration option isn't available.

## Task 4: Verify near real-time replication

1. In SSMS, **Connect** > **Database Engine** to the SQL MI FQDN, **SQL Server Authentication**, login `contosoadmin` / `IAE5fAijit0w^rDM`, check **Trust server certificate**.

1. On the **<inject key="SQLVM Name" enableCopy="false"/>** connection, run:

   ```sql
   USE WideWorldImporters;
   GO
   INSERT [dbo].[Game] (Title, Description, Rating, IsOnlineMultiplayer)
   VALUES ('Space Adventure', 'Explore the universe with our newest online multiplayer gaming experience.', 'T', 1);
   ```

1. On the **SQL MI** connection, run (replace **SUFFIX** with **<inject key="Suffix" />**):

   ```sql
   USE WideWorldImportersSUFFIX;
   GO
   SELECT Title FROM dbo.Game WHERE Title = 'Space Adventure';
   SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability') AS Updateability;
   ```

1. Observe the **Space Adventure** row is already on SQL MI and **Updateability** is **READ_ONLY**.

## Task 5: Cut over to SQL MI

1. On the **<inject key="SQLVM Name" enableCopy="false"/>** connection, simulate stopping writes:

   ```sql
   USE master;
   GO
   ALTER DATABASE WideWorldImporters SET READ_ONLY WITH ROLLBACK IMMEDIATE;
   ```

1. In the portal, return to **Database migration** > **Migrate**, select the migration, confirm lag is **0**, and select **Complete cutover**. Check the confirmation box (SQL 2016 doesn't support failback) and select **Complete cutover**.

1. Select **Refresh** until the status is **Succeeded**.

1. On the **SQL MI** connection, run (replace **SUFFIX**):

   ```sql
   USE WideWorldImportersSUFFIX;
   GO
   SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability') AS Updateability;  -- now READ_WRITE
   ```

1. Set the source back to read-write:

   ```sql
   USE master;
   GO
   ALTER DATABASE WideWorldImporters SET READ_WRITE WITH ROLLBACK IMMEDIATE;
   ```

## Summary

You used Azure Arc and the Managed Instance link to replicate the database to SQL MI, confirmed near-real-time replication, and cut over with only seconds of downtime.

### You have successfully completed the exercise. Now click on **Next >>** from the lower right corner to move on to the next exercise.
