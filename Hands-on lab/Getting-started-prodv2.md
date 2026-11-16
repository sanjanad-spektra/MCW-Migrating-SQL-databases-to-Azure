# Migrating SQL Databases to Azure SQL Managed Instance with Azure Arc

## 📖 Overview

Welcome to the **Migrating SQL Databases to Azure SQL Managed Instance with Azure Arc** lab.

In this lab, you will migrate an on-premises **SQL Server 2016** database (`WideWorldImporters`) to **Azure SQL Managed Instance**, using modern, fully supported migration tooling. You will begin by connecting your SQL Server to Azure with **Azure Arc** and running a **migration assessment** to confirm the database is ready for Azure SQL Managed Instance.

You will then migrate the database using **two complementary paths**: an online, near-zero-downtime migration with the **Azure Arc + Managed Instance link** (distributed availability groups), and a backup-based migration with **Azure Database Migration Service (DMS)** reading backups directly from **Azure Blob Storage** — with no file share or self-hosted integration runtime required.

Finally, you will modernize and secure the migrated database on Azure SQL Managed Instance by publishing an application with **regional virtual network integration**, comparing performance with **Query Store**, and applying built-in security capabilities including **Data Discovery & Classification**, **Microsoft Defender for SQL**, **Transparent Data Encryption (TDE)**, **Dynamic Data Masking**, and **read scale-out**.

By the end of this lab, you will understand how to assess, migrate, and secure a SQL Server 2016 database on Azure SQL Managed Instance using the latest Azure Arc-based tooling.

## 🎯 Objectives

In this lab, you will:

- Connect a SQL Server 2016 instance to Azure with **Azure Arc** and run a **migration assessment**

- Migrate the database with the **Azure Arc + Managed Instance link** (Path A — minimal downtime)

- Migrate the database with **Azure Database Migration Service** from **Azure Blob Storage** (Path B)

- Publish an application and connect it to Azure SQL Managed Instance using **regional VNet integration**, and compare performance with **Query Store**

- Secure the database with **Data Discovery & Classification**, **Microsoft Defender for SQL**, and **Transparent Data Encryption**

- Apply **Dynamic Data Masking** and configure **read scale-out**

## ⚙️ Prerequisites

Participants should have basic knowledge of the following:

- SQL Server administration fundamentals (databases, backups, recovery models)

- Azure fundamentals (resource groups, virtual networks, the Azure portal)

- Familiarity with **SQL Server Management Studio (SSMS)**

> **Note:** The lab environment is preconfigured with a SQL Server 2016 virtual machine, the source `WideWorldImporters` database, a shared Azure SQL Managed Instance, and the required networking and permissions.

## 🏗️ Architecture

## 🔍 Explanation of Components

- **SQL Server 2016 (Azure Arc-enabled):** the source database server. Azure Arc projects it into Azure for inventory, migration assessment, and the Managed Instance link.

- **Azure Arc:** provides the **migration assessment** and the **Managed Instance link** migration experience from the Azure portal (replacing the retired Data Migration Assistant).

- **Azure SQL Managed Instance:** the fully managed PaaS target, with near 100% SQL Server compatibility.

- **Azure Database Migration Service (DMS):** performs the backup-based migration (Path B) by restoring backups from Azure Blob Storage to the managed instance.

- **Azure Blob Storage:** stores the database backups written directly from SQL Server with `BACKUP ... TO URL`.

- **Built-in security (Defender for SQL, Data Discovery & Classification, TDE, Dynamic Data Masking):** used after migration to assess vulnerabilities, classify sensitive data, encrypt the database, and mask sensitive columns.

## 🚀 Getting Started with the lab
 
A preconfigured environment has been prepared for you to explore **Azure Arc** and **Azure SQL Managed Instance** and learn how to assess, migrate, and secure a SQL Server 2016 database.
 
## Accessing Your Lab Environment
 
Once you're ready to dive in, your virtual machine and **Guide** will be right at your fingertips within your web browser.

![](./media/gs-g-et-1.png)

## Virtual Machine & Lab Guide
 
Your virtual machine is your workhorse throughout the workshop. The lab guide is your roadmap to success.
 
## Exploring Your Lab Resources
 
To get a better understanding of your lab resources and credentials, navigate to the **Environment** tab.

   ![](./media/cor-op-rt-g-3.png)
 
## Utilizing the Split Window Feature
 
For convenience, you can open the lab guide in a separate window by selecting the **Split Window** button from the Top right corner.
 
   ![](./media/cor-op-rt-g-4.png)
 
## Managing Your Virtual Machine
 
Feel free to start, stop, or restart your virtual machine as needed from the **Resources** tab. Your experience is in your hands!
 
  ![](./media/cor-op-rt-g-6.png)

## Lab Guide Zoom In/Zoom Out
 
To adjust the zoom level for the environment page, click the **A↕ : 100%** icon located next to the timer in the lab environment.

   ![](media/mcp-image150.png)

## Resize the Virtual Machine View

Use the **slider (three vertical dots)** located between the **Virtual Machine** and the **Lab Guide** panes to adjust the display size, allowing you to customize the layout based on your preference.

   ![](media/pmcp-image151.png)
 
## 📞 Support Contact
 
The **CloudLabs support team** is available 24/7, 365 days a year, via email and live chat to ensure seamless assistance at any time. We offer dedicated support channels tailored specifically for both learners and instructors, ensuring that all your needs are promptly and efficiently addressed.

Learner Support Contacts:

- Email Support: cloudlabs-support@spektrasystems.com

- Live Chat Support: https://cloudlabs.ai/labs-support

Now, click on **Next** from the lower right corner to move on to the next page.

  ![](./media/gettingstarted/next-1207.png)

### Happy Learning!!
