## Exercise 4: Connect the web application and compare performance

**Duration: 40 Minutes**

### Overview

In this exercise, you will publish the WideWorldImporters web application to Azure App Service and point it at the database you migrated with the Managed Instance link (Path A). Because SQL Managed Instance has a private IP address inside its virtual network, you will connect the app to that network with **regional VNet integration** on a subnet delegated to `Microsoft.Web/serverFarms` — the modern replacement for the gateway-based point-to-site connection. Finally, you will use **Query Store** to compare the workload's performance on SQL MI with the baseline captured on SQL Server 2016.

### Objectives

- **Task 1:** Deploy the web app to Azure
- **Task 2:** Update the App Service connection strings
- **Task 3:** Configure regional VNet integration and open the web app
- **Task 4:** Compare performance with Query Store (optional)

### Task 1: Deploy the web app to Azure

In this task, you will use Visual Studio 2022 on the JumpBox VM to publish the `WideWorldImporters` web application to Azure App Service.

1. On the **JumpBox** VM, in File Explorer, navigate to `C:\hands-on-lab\MCW-Migrating-SQL-databases-to-Azure-master\Hands-on lab\lab-files`. Double-click **WideWorldImporters.sln** to open the solution in Visual Studio.

   ![The WideWorldImporters.sln solution is highlighted.](media/windows-explorer-lab-files-web-solution.png "Windows Explorer")

1. If prompted, select **Visual Studio 2022** and select **OK**.

1. Select **Sign in** > **Work or School account** and enter the following Azure credentials when prompted:

   * Email/Username: <inject key="AzureAdUserEmail"></inject>
   * Password: <inject key="AzureAdUserPassword"></inject>

1. Once signed in, select **Start Visual Studio**. If a security warning appears, uncheck **Ask me for every project in this solution** and select **OK**.

1. In **Solution Explorer**, right-click the **WideWorldImporters.Web** project and select **Publish**.

   ![The Publish context menu item is highlighted.](media/visual-studio-project-publish.png "Visual Studio")

1. On the **Publish** dialog, select **Azure** and select **Next**.

1. For **Specific target**, select **Azure App Service (Windows)** and select **Next**.

1. Under **App Service**, select your subscription, expand the **<inject key="Resource Group Name" enableCopy="false"/>** resource group, select the **wwi-web-<inject key="Suffix" enableCopy="false"/>** Web App, and select **Finish**. When the profile is created, select **Close**.

1. Select **Publish**. When it completes, the Output window shows **Publish Succeeded**.

   ![The Publish Succeeded message is displayed.](media/visual-studio-output-publish-succeeded.png "Visual Studio")

   > If you open the published site now, you get an error page because the connection strings don't point to SQL MI yet. You fix this next.

### Task 2: Update the App Service connection strings

In this task, you will update the App Service connection strings to point the application at the migrated database on SQL Managed Instance.

1. In the Azure portal, open the **<inject key="Resource Group Name" enableCopy="false"/>** resource group and select the **wwi-web-<inject key="Suffix" enableCopy="false"/>** App Service.

1. Under **Settings**, select **Environment variables**, select the **Connection strings** tab, and select **Advanced edit**.

1. Set the value of both `wwiContext` and `WwiReadOnlyContext` to the following:

    ```
    Server=tcp:your-sqlmi-host-fqdn-value,1433;Database=WideWorldImportersSUFFIX;User ID=contosoadmin;Password=IAE5fAijit0w^rDM;Encrypt=True;TrustServerCertificate=False;
    ```

1. Replace `your-sqlmi-host-fqdn-value` with the SQL MI FQDN from Exercise 2, and replace `SUFFIX` with **<inject key="Suffix" />**. Select **OK**.

   >**Note:** The application reads its connection strings with `GetConnectionString()`, so you only set them under **Connection strings**. `Encrypt=True;TrustServerCertificate=False` makes the app validate the managed instance's certificate.

1. Select **Apply**, then **Confirm** when told the app will restart.

### Task 3: Configure regional VNet integration and open the web app

In this task, you will configure regional VNet integration so the web app can reach the managed instance's private IP address, then open the running site. This replaces the gateway-based point-to-site connection used in earlier versions of the lab.

SQL MI has only a private IP address in its virtual network. Regional VNet integration routes the web app's outbound traffic into that network. It needs a **subnet delegated to `Microsoft.Web/serverFarms`** — the lab environment provides one in the `sqlmi-vnet` virtual network.

1. On the **wwi-web-<inject key="Suffix" enableCopy="false"/>** App Service, under **Settings**, select **Networking**. Under **Outbound traffic configuration**, select **Not configured** next to **Virtual network integration**.

1. Select **Add virtual network integration**.

1. Enter the following and select **Connect**:

   - **Virtual Network**: **`sqlmi-vnet`** (the SQL MI virtual network)
   - **Subnet**: the subnet **delegated to `Microsoft.Web/serverFarms`** (in this environment it's named **`vnet-sqlmi--cus`**)

   > **Important:** Don't select the `ManagedInstance` subnet or any subnet already in use. Regional VNet integration only works with a subnet delegated to `Microsoft.Web/serverFarms`.

1. Wait a few minutes for the integration to be added and the app to restart, then select **Refresh** to confirm the network is connected.

1. Select **Overview**, then select the **Default domain** link to open the website. Verify it loads and shows data from SQL MI.

   ![The WideWorldImporters web app home page.](media/wwi-web-app.png "WideWorldImporters Web")

   >**Note:** The network change can take a few minutes to reach the web app. If you see an error, refresh a few times, or select **Restart** on the web app's toolbar.

### Task 4: Compare performance with Query Store (optional)

In this task, you will run the same workload on SQL Managed Instance and use Query Store to compare performance before and after the migration.

Query Store data is stored inside the database, so the baseline you captured on SQL Server 2016 in Exercise 1 moved to SQL MI with the database.

> **Note:** Do this task only if you completed **Exercise 1, Task 2**.

1. On the **JumpBox** VM, in SSMS, select the **SQL MI** connection, select **New Query**, paste the following (replace **SUFFIX** with **<inject key="Suffix" />**), and select **Execute**:

    ```sql
    USE WideWorldImportersSUFFIX;
    GO

    SELECT TOP (50) GamerId, GamerScore, GamesPlayed
    FROM dbo.Leaderboard
    ORDER BY GamerScore DESC;
    GO 50

    SELECT Title, Rating
    FROM dbo.Game
    WHERE IsOnlineMultiplayer = 1
    ORDER BY Title;
    GO 50
    ```

1. In Object Explorer, under the SQL MI connection, expand **Databases** > **WideWorldImporters<inject key="Suffix" enableCopy="false"/>** > **Query Store**, and double-click **Top Resource Consuming Queries**.

1. Select **Configure**, set the time interval to cover both runs, and select **OK**. Select the Leaderboard query and compare the average **duration** and **CPU time** between SQL Server 2016 (before) and SQL MI (after).

1. The migrated database keeps compatibility level **130** (SQL Server 2016). Run the following to raise it, then run the workload again and check **Regressed Queries** under Query Store:

    ```sql
    SELECT name, compatibility_level FROM sys.databases WHERE name = DB_NAME();
    GO
    ALTER DATABASE CURRENT SET COMPATIBILITY_LEVEL = 160;
    GO
    ```

   > Query Store lets you compare performance before and after migration with one tool. If a query regresses after the compatibility change, you can force its earlier plan from the plan summary pane until it's fixed.

## Summary

In this exercise, you published the web application, connected it to the migrated database through regional VNet integration, and used Query Store to compare the workload's performance before and after the migration.

### You have successfully completed the exercise. Now click on **Next >>** from the lower right corner to move on to the next exercise.
