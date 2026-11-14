## Exercise 4: Connect the web application and compare performance

### Estimated Duration: 40 Minutes

In this exercise, you publish the WideWorldImporters web application to Azure App Service and point it at the database you migrated with the Managed Instance link (Path A). You connect the app to the MI's network with **regional VNet integration** on a delegated subnet, then use **Query Store** to compare performance on SQL MI with the baseline from Exercise 1.

## Lab Objectives

- Task 1: Deploy the web app to Azure
- Task 2: Update the App Service connection strings
- Task 3: Configure regional VNet integration and open the web app
- Task 4: Compare performance with Query Store (optional)

## Task 1: Deploy the web app to Azure

1. On the **JumpBox** VM, in File Explorer, go to `C:\hands-on-lab\MCW-Migrating-SQL-databases-to-Azure-master\Hands-on lab\lab-files`. Double-click **WideWorldImporters.sln** to open it in Visual Studio.

1. If prompted, select **Visual Studio 2022** and **OK**.

1. Select **Sign in** > **Work or School account** and enter the Azure credentials if prompted:

   * Email/Username: <inject key="AzureAdUserEmail"></inject>
   * Password: <inject key="AzureAdUserPassword"></inject>

1. Once signed in, select **Start Visual Studio**. If a security warning appears, uncheck **Ask me for every project in this solution** and **OK**.

1. In **Solution Explorer**, right-click **WideWorldImporters.Web** > **Publish**.

1. On **Publish**, select **Azure** > **Next**.

1. For **Specific target**, select **Azure App Service (Windows)** > **Next**.

1. Under **App Service**, select your subscription, expand **<inject key="Resource Group Name" enableCopy="false"/>**, select **wwi-web-<inject key="Suffix" enableCopy="false"/>**, and **Finish** > **Close**.

1. Select **Publish**. When done, the Output window shows **Publish Succeeded**.

   > Opening the site now returns an error — the connection strings don't point to SQL MI yet. You fix this next.

## Task 2: Update the App Service connection strings

1. In the portal, open **<inject key="Resource Group Name" enableCopy="false"/>** > **wwi-web-<inject key="Suffix" enableCopy="false"/>**.

1. Under **Settings**, select **Environment variables** > **Connection strings** tab > **Advanced edit**.

1. Set the value of both `wwiContext` and `WwiReadOnlyContext` to:

    ```
    Server=tcp:your-sqlmi-host-fqdn-value,1433;Database=WideWorldImportersSUFFIX;User ID=contosoadmin;Password=IAE5fAijit0w^rDM;Encrypt=True;TrustServerCertificate=False;
    ```

1. Replace `your-sqlmi-host-fqdn-value` with the SQL MI FQDN from Exercise 2, and `SUFFIX` with **<inject key="Suffix" />**. Select **OK**.

   >**Note:** The app reads its connection strings with `GetConnectionString()`, so you only set them under **Connection strings**.

1. Select **Apply**, then **Confirm**.

## Task 3: Configure regional VNet integration and open the web app

SQL MI has only a private IP. Regional VNet integration routes the app's outbound traffic into the MI's network, using an **empty subnet delegated to `Microsoft.Web/serverFarms`** (the environment provides **webapp-subnet**).

1. On **wwi-web-<inject key="Suffix" enableCopy="false"/>**, under **Settings** > **Networking** > under **Outbound traffic configuration**, select **Not configured** next to **Virtual network integration**.

1. Select **Add virtual network integration**.

1. Enter the following and **Connect**:

   - **Virtual Network**: the SQL MI virtual network
   - **Subnet**: **webapp-subnet**

   > **Important:** Don't select the managed-instance subnet or any subnet already in use. Regional VNet integration needs an empty subnet delegated to `Microsoft.Web/serverFarms`.

1. Wait a few minutes, then **Refresh** to confirm the network is connected.

1. Select **Overview** > **Default domain** to open the website. Verify it loads with data from SQL MI.

   >**Note:** The network change can take a few minutes. If you see an error, refresh a few times or **Restart** the web app.

## Task 4: Compare performance with Query Store (optional)

> Do this only if you completed **Exercise 1, Task 2**.

1. On the **JumpBox** VM, in SSMS on the **SQL MI** connection, run the same workload (replace **SUFFIX**):

    ```sql
    USE WideWorldImportersSUFFIX;
    GO
    SELECT TOP (50) GamerTag, Name, GamerScore, GamesPlayed
    FROM dbo.Leaderboard ORDER BY GamerScore DESC;
    GO 50
    SELECT Title, Rating FROM dbo.Game WHERE IsOnlineMultiplayer = 1 ORDER BY Title;
    GO 50
    ```

1. In Object Explorer, expand **WideWorldImporters<inject key="Suffix" enableCopy="false"/>** > **Query Store** > **Top Resource Consuming Queries**. Compare the Leaderboard query's duration/CPU before (SQL 2016) and after (SQL MI).

1. The migrated database keeps compatibility level **130**. Raise it and re-run the workload, then check **Regressed Queries**:

    ```sql
    ALTER DATABASE CURRENT SET COMPATIBILITY_LEVEL = 160;
    GO
    ```

## Summary

You published the web application, connected it to the migrated database through regional VNet integration, and used Query Store to compare performance before and after the migration.

### You have successfully completed the exercise. Now click on **Next >>** from the lower right corner to move on to the next exercise.
