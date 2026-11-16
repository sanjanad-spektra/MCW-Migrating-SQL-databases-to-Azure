## Exercise 6: Enable Dynamic Data Masking and use read scale-out

**Duration: 30 Minutes**

### Overview

In this exercise, you will protect sensitive columns with **Dynamic Data Masking (DDM)**, which hides sensitive data in query results from users without privilege to see it, and then route the web app's read-only reporting queries to a secondary replica with **read scale-out**.

### Objectives

- **Task 1:** Enable DDM on credit card numbers
- **Task 2:** Apply DDM to email addresses
- **Task 3:** Route read-only queries to a secondary replica with read scale-out

## Task 1: Enable DDM on credit card numbers

In this task, you will apply a dynamic data mask to the credit card number column and confirm a non-privileged user sees only the last four digits.

1. On the **JumpBox** VM, in SSMS on the **SQL MI** connection, expand **Databases** > **WideWorldImporters<inject key="Suffix" enableCopy="false"/>** > **Tables**, right-click **Sales.CreditCard** > **Select Top 1000 Rows**. Note `CardNumber` is in plain text.

1. Create a test user (replace **SUFFIX**):

   ```sql
   USE [WideWorldImportersSUFFIX];
   GO
   CREATE USER DDMUser WITHOUT LOGIN;
   GRANT SELECT ON [Sales].[CreditCard] TO DDMUser;
   ```

1. Run as the test user — card numbers still in plain text:

   ```sql
   USE [WideWorldImportersSUFFIX];
   GO
   EXECUTE AS USER = 'DDMUser';
   SELECT TOP 10 * FROM [Sales].[CreditCard];
   REVERT;
   ```

1. Add a mask to the `CardNumber` column:

   ```sql
   USE [WideWorldImportersSUFFIX];
   GO
   ALTER TABLE [Sales].[CreditCard]
   ALTER COLUMN [CardNumber] NVARCHAR(25) MASKED WITH (FUNCTION = 'partial(0,"xxx-xxx-xxx-",4)');
   ```

1. Run the test-user query again — only the last four digits show.

## Task 2: Apply DDM to email addresses

In this task, you will apply the built-in email mask to the login email column and confirm it is masked for a non-privileged user.

1. Grant access and mask `LoginEmail` in `dbo.Gamer` with the built-in `email()` function (replace **SUFFIX**):

   ```sql
   USE [WideWorldImportersSUFFIX];
   GO
   GRANT SELECT ON [dbo].[Gamer] TO DDMUser;
   GO
   ALTER TABLE [dbo].[Gamer]
   ALTER COLUMN [LoginEmail] NVARCHAR(250) MASKED WITH (FUNCTION = 'email()');
   ```

1. Run as the test user and look at `LoginEmail`:

   ```sql
   USE [WideWorldImportersSUFFIX];
   GO
   EXECUTE AS USER = 'DDMUser';
   SELECT TOP 10 * FROM [dbo].[Gamer];
   REVERT;
   ```

   > The `email()` function shows the first letter and replaces the rest with `XXX@XXXX.com`.

## Task 3: Route read-only queries to a secondary replica with read scale-out

In this task, you will route the web application’s read-only reporting queries to a read-only secondary replica using read scale-out.

Each database in the SQL MI **Business Critical** tier has built-in replicas. **Read scale-out** lets read-only connections use one, at no extra cost, by adding `ApplicationIntent=ReadOnly` to the connection string.

> **Note:** Read scale-out is available only in the **Business Critical** service tier (not General Purpose).

1. Open the web app from **wwi-web-<inject key="Suffix" enableCopy="false"/>** (**Default domain**) and select **Leaderboard**. The page shows `READ_WRITE` — the `Updateability` property of the connected database.

1. In the portal, on the App Service, select **Environment variables** > **Connection strings** > **WwiReadOnlyContext**.

1. Add to the end of the **Value**, and **OK**:

   ```
   ApplicationIntent=ReadOnly;
   ```

1. Select **Apply**, then **Confirm**.

1. Return to the web app and refresh the **Leaderboard** page — it now shows `READ_ONLY`, meaning the reporting queries run on a read-only replica.

## Summary

You masked sensitive columns with Dynamic Data Masking and used read scale-out to run the leaderboard report on a read-only replica.

## You have successfully completed the lab!

In this **Data Modernization** lab, you migrated an end-of-support SQL Server 2016 database to Azure SQL Managed Instance with current tools: you assessed the server with **Azure Arc**, migrated with both the **Managed Instance link** and **Azure DMS from Blob Storage**, connected the application through **regional VNet integration**, compared performance with **Query Store**, and secured and scaled the database with **Microsoft Defender for SQL**, **Data Discovery & Classification**, **TDE**, **Dynamic Data Masking** and **read scale-out**.
