# Session 2: Storage Account and ADF (Study guide)

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)

**What you will learn:** You will understand what an Azure Storage Account is and the difference between Blob storage and Azure Data Lake Storage Gen2 (ADLS Gen2). You will learn how Azure organises resources with subscriptions, resource groups and regions, why companies separate OLTP and OLAP systems, and what an end-to-end data engineering architecture looks like. You will create an ADLS Gen2 account, a Blob storage account and an Azure Data Factory, and take a tour of ADF Studio.

**Before you start:** Your free Azure account from Session 1, Google Chrome on a laptop or desktop, and a small test file (a CSV file is ideal).

**Slides:** [slides.md](slides.md) · [Download the slides as PDF](slides.pdf)

> **Names used in this guide**
>
> Storage account and Data Factory names must be **unique across all of Azure**, so add your initials or a number to the names below.
>
> | Item | Name in this guide | Rule |
> | --- | --- | --- |
> | Resource group | `rg-session02-lab` | Unique within your subscription |
> | ADLS Gen2 account | `stsession02adls` | 3 to 24 lowercase letters and numbers, globally unique |
> | Blob storage account | `stsession02blob` | Same rules as above |
> | Data Factory | `adf-session02-lab` | Globally unique; letters, numbers and hyphens |
> | Container | `landing` | Lowercase, 3 to 63 characters |

## Recap of Session 1

- **Cloud computing** is using computing resources over the internet, on demand, and paying for what you use.
- **Scalability and autoscaling**, **geo distribution**, **agility** and **pay-as-you-go** are its main benefits; cost creep, internet dependence and vendor lock-in are among its cons.
- You created your **free Azure account**. Today it gives you the **subscription** that your resources are created in.

How Session 1 connects to today: geo distribution becomes the **region** you choose for each resource, and pay-as-you-go is why we choose low-cost options and set a budget alert.

## 1. Azure Storage Account

### In one sentence

An Azure Storage Account is a Microsoft-managed container for cloud storage services: it holds your files and data safely, and you pay for what you store and use.

### What is inside a storage account

| Service | What it stores | Who mainly uses it |
| --- | --- | --- |
| **Blob service** | Files (objects) of any type: CSV, JSON, Parquet, images, videos, backups | Developers and data engineers |
| **ADLS Gen2** | Blob storage plus real folders and folder-level security, built for analytics | **Data engineers: the data lake** |
| **Queue** | Messages passed between parts of an application | Developers |
| **File share** | Network drives (SMB or NFS) you can mount like a drive letter | IT and application teams |
| **Tables** | Simple NoSQL key-value data | Developers |

As a data engineer you will mostly use **Blob** and, above all, **ADLS Gen2**.

### An easy analogy

Think of a personal cloud drive such as OneDrive or Google Drive: you upload any file, organise it in folders and reach it from anywhere. Azure Blob and ADLS Gen2 work the same way, but at a much larger scale, and they are mainly used by programs and data services (Data Factory, Databricks) rather than by people clicking in a browser. On AWS the equivalent is **Amazon S3**; on Google Cloud it is **Cloud Storage**.

### Account kind: StorageV2

In the portal, the **Kind** column shows **StorageV2 (general-purpose v2)**, the standard kind for almost everything. A Blob account and an ADLS Gen2 account are **both StorageV2**. There is no separate "ADLS" resource: one setting, **hierarchical namespace**, turns a StorageV2 account into ADLS Gen2.

**Think about it:** Which cloud file services do you already use every day, and what do you store in them?

## 2. Subscription, resource group and region

Every **Create** form in Azure first asks three questions: which subscription pays for the resource, which resource group it belongs to, and which region it lives in.

### The Azure hierarchy

![The Azure resource hierarchy](slides/slide-12.jpg)

- **Tenant:** your organisation's Microsoft Entra ID directory, with its users and groups.
- **Management groups (optional):** group many subscriptions to apply policies to all of them.
- **Subscription:** the **billing and access boundary**. Every resource is billed to exactly one subscription.
- **Resource group:** a logical folder for related resources.
- **Resources:** storage accounts, data factories, databases and so on.

Permissions and policies flow **down** the hierarchy: if you have the Contributor role on a resource group, you can manage everything inside it.

### Subscription

The subscription answers two questions: **who pays** for this resource, and **which access rules** apply to it. Yours may be called *Azure subscription 1* or *Free Trial*. Companies often keep separate subscriptions for production and non-production, or per department.

### Resource group

A **resource group** is a container that holds related resources for an Azure solution.

![Why resource groups exist](slides/slide-14.jpg)

- A company runs many projects, and each project has **Dev**, **UAT** and **Prod** environments with their own resources.
- A common pattern is **one resource group per project per environment**, for example `rg-a-dev`, `rg-a-uat`, `rg-a-prod`.
- A resource group can hold **many resource types**, and the resources can be in **different regions**.
- Every resource belongs to **exactly one** resource group (you can move it later).
- You can apply **access control (IAM)**, **tags** and **cost reports** to the whole group.
- **Deleting a resource group deletes everything inside it.** It is the easiest way to clean up a lab.

### Region

A **region** is a geographic area that contains one or more Azure datacenters, for example East US, West Europe, South Africa North or South India.

| Choose a region for | Why |
| --- | --- |
| **Latency** | Keep data close to its users and to the services that process it |
| **Compliance** | Some laws require data to stay in a country or geography |
| **Cost** | Prices differ slightly between regions |
| **Availability** | Not every service or feature exists in every region |

- Most regions have a **paired region** in the same geography (for example South India and Central India). Azure uses the pair for geo-redundant copies.
- Keep services that work together (storage, Data Factory, Databricks) in the **same region**.

> **Tip**
>
> Use a region close to you. The live demo used **South India** for storage and **East US** for Data Factory; in your own lab, put all of today's resources in the same region.

**Think about it:** If your users are in Rwanda, which region would you choose, and what would you check first?

## 3. Create an ADLS Gen2 account

In this lab you create your data lake.

### Step by step

1. In the portal, search for **Storage accounts** and select **+ Create**.
2. **Subscription:** select your subscription.
3. **Resource group:** select **Create new**, enter `rg-session02-lab` and select **OK**. The field now shows **(New) rg-session02-lab**.
4. **Storage account name:** `stsession02adls` plus your initials.
5. **Region:** choose a region close to you.
6. **Performance:** **Standard**.
7. **Redundancy:** **Locally-redundant storage (LRS)**, the lowest-cost option for a lab.
8. Select **Next** to open the **Advanced** tab.
9. Tick **Enable hierarchical namespace**. This one checkbox makes the account ADLS Gen2.
10. Leave **Minimum TLS version** at **1.2** and leave the other tabs on their defaults.
11. Select **Review + create**, check the summary (look for **Enable hierarchical namespace: Enabled**), then select **Create**.
12. When you see **Your deployment is complete**, select **Go to resource**.

![Enable hierarchical namespace](slides/slide-22.jpg)

### Settings explained

**Performance**

- **Standard:** recommended for most scenarios (general-purpose v2).
- **Premium:** very low latency on SSDs, at a higher cost. Rarely needed for a data lake.

**Redundancy:** how many copies Azure keeps, and where.

| Option | Copies | Survives |
| --- | --- | --- |
| **LRS** (locally redundant) | 3 copies in one datacenter | A disk or rack failure |
| **ZRS** (zone redundant) | 3 copies across availability zones | A datacenter failure |
| **GRS** (geo-redundant) | LRS in the primary region plus LRS in the paired region | A regional disaster |
| **RA-GRS** | GRS with read access to the secondary region | A regional disaster, and reads continue |
| **GZRS / RA-GZRS** | ZRS in the primary region plus a geo copy | The highest durability |

The live demo used **RA-GRS**, so the account overview showed *Primary: South India, Secondary: Central India*.

**Hierarchical namespace:** the portal describes it as enabling "file and directory semantics", accelerating "big data analytics workloads" and enabling "access control lists (ACLs)". In short: real folders, faster analytics and fine-grained security.

- An existing Blob account can be upgraded to ADLS Gen2 later, but **once hierarchical namespace is on, it cannot be turned off**.

**Security defaults** (keep them in real projects):

- **Secure transfer required:** only HTTPS connections.
- **Blob anonymous access: Disabled:** nobody on the internet can read your files without credentials.
- **Minimum TLS version 1.2.**
- **Access tier: Hot:** for data you read often. Cool, Cold and Archive are cheaper to store but cost more to read.

**View automation template:** everything you choose in the portal becomes an ARM template (infrastructure as code). In companies, resources are often created from templates such as ARM, Bicep or Terraform instead of by clicking.

**Tags:** Name:Value labels such as `env = dev` or `owner = yourname`. They do not change how a resource works, but they drive cost reports and governance.

### Create a container and upload a file

1. In your storage account, open **Data storage** > **Containers**. The `$logs` container is created by Azure for logs.
2. Select **+ Add container**, enter `landing` and select **Create**. The access level is **Private** because anonymous access is disabled.
3. Open **landing** and select **Upload**. Browse for your small test file and select **Upload**.
4. Select **+ Add Directory**, enter `abc` and select **Save**. Leave the folder empty: you compare it with Blob storage in section 9.

- A **container** is the top-level folder of a storage account (in ADLS it is also called a **file system**).
- **landing** is a common name for the **raw landing zone**, where ingested files first arrive, untouched.
- **Authentication method: Access key** means the portal uses the account key, which works like a master password. In real projects, Microsoft Entra ID with role-based access control is preferred.

> **Watch out**
>
> Never share or commit storage account **access keys** or **connection strings**. Anyone with the key has full access to the account.

### Set a budget alert

As promised in Session 1, protect yourself from surprise bills:

1. Search for **Cost Management** and open **Budgets**.
2. Select **+ Add**, choose your subscription as the scope, and enter a small monthly amount (for example 10 USD).
3. Add an alert at **80%** of the budget, send it to your email, and select **Create**.

**Think about it:** What else would you store in a `landing` container for a company you know?

## 4. OLTP

### The application database

![OLTP: the application database](slides/slide-32.jpg)

An online shop's customers use a mobile app, a website and a desktop app. All of them read and write to one database: the **OLTP (Online Transaction Processing)** database.

- Built for **many small, fast transactions**: place an order, update an address, show one basket.
- **Read fast and write fast**, but only **small amounts of data** per operation.
- **Normalised** tables (data split into many related tables) and designed for **many users at the same time**.
- Examples: Oracle, MySQL, SQL Server, PostgreSQL, Azure SQL Database.

### The problem: analytics on OLTP

The business now wants insights, for example total sales per region per month for the last three years. Data analysts and BI (Business Intelligence) teams write **complex queries**: large scans, joins and aggregations.

If those heavy queries run on the same OLTP database, it becomes **overloaded**, the apps become **slow for customers**, and the business loses sales.

> **Golden rule**
>
> Never run heavy analytics on the production OLTP database.

## 5. OLAP

The solution is to keep OLTP for the applications and copy the data into a separate system designed for analysis: **OLAP (Online Analytical Processing)**, usually a **data warehouse**.

![The solution: a separate OLAP system](slides/slide-34.jpg)

- **Internal users** (BI developers and data analysts) query the warehouse, not the application database.
- The apps stay fast, and analytics becomes fast too, because a warehouse is designed for large scans over years of history (denormalised star schemas and columnar storage).
- Examples: Azure Synapse Analytics, Microsoft Fabric Warehouse, Oracle data warehouse, IBM Db2 Warehouse.

## 6. OLTP vs OLAP

|   | OLTP | OLAP |
| --- | --- | --- |
| **Full name** | Online Transaction Processing | Online Analytical Processing |
| **Purpose** | Run the business | Analyse the business |
| **Users** | Customers and applications | Analysts, BI teams, data scientists |
| **Operations** | Many small INSERT, UPDATE and single-row reads | Few large, complex SELECT queries |
| **Data per query** | Small (rows) | Huge (millions of rows or more) |
| **Data age** | Current | Historical (years) |
| **Design** | Normalised (3NF) | Denormalised (star schema) |
| **Examples** | Oracle, MySQL, SQL Server, PostgreSQL | Synapse, Fabric Warehouse, Oracle DW, Db2 Warehouse |

### ETL: the bridge between them

**ETL jobs** copy data from OLTP to OLAP:

- **E, Extract:** read data from the source systems.
- **T, Transform:** clean, remove duplicates, fix data types, join, aggregate and apply business rules.
- **L, Load:** write the result into the warehouse or data lake.

A modern variation is **ELT**: load the raw data into the lake first, then transform it there with engines such as Spark.

> **What a data engineer does**
>
> A data engineer builds and runs the pipelines that move and transform data from where it is created to where it is analysed.

**Think about it:** An ATM withdrawal and a dashboard of withdrawals per city over five years: which one is OLTP and which one is OLAP?

## 7. End-to-end data engineering architecture

In the cloud the OLTP, ETL, OLAP idea stays the same, but with many more sources and services.

![Reference architecture](slides/slide-40.jpg)

1. **Sources:** a real company has data in many systems, for example orders in an Oracle database, HR data in SQL Server, customers in a CRM, shipping in a SaaS platform, web traffic in an analytics tool and partner data behind a REST API.
2. **Ingestion:** pipelines built with **Azure Data Factory** copy data from every source into one place.
3. **Storage:** that place is the data lake, **ADLS Gen2**, organised in zones such as **landing (raw)** and **curated**.
4. **Processing:** **Azure Databricks** with **PySpark** (Python on Apache Spark) cleans and arranges the raw data into curated tables.
5. **Serving:** the curated data is used by **BI dashboards** (for example Power BI), **data analysts** and **AI or machine learning** teams.

ADLS Gen2, Databricks and the pipelines together form the **data platform** that a data engineer builds and maintains.

**Example:** an order placed in the mobile app is stored in the Oracle database. Tonight, Data Factory copies the new orders to `landing/orders/`. Databricks cleans them and joins them with customer data. Tomorrow morning, the sales dashboard shows yesterday's revenue.

This bootcamp builds a small version of it: **Azure SQL Database → Azure Data Factory → ADLS Gen2 → Databricks**.

## 8. Blob vs ADLS Gen2

![Flat vs hierarchical namespace](slides/slide-42.jpg)

- **Blob storage** has a **flat namespace**: a "folder" is only part of the file name, such as `landing/abc/file1.csv`. The portal calls it a **virtual directory**.
- **ADLS Gen2** has a **hierarchical namespace**: directories are real objects.

Why this matters for analytics: big-data jobs constantly list, rename and delete folders. In Blob storage, renaming a folder with 10,000 files means copying and deleting 10,000 files. In ADLS Gen2 it is one instant, **atomic** operation.

| Feature | Blob (hierarchical namespace off) | ADLS Gen2 (hierarchical namespace on) |
| --- | --- | --- |
| **Account kind** | StorageV2 | StorageV2 (the same) |
| **Namespace** | Flat: folders are name prefixes | Hierarchical: real directories |
| **Empty folder** | Disappears | Stays |
| **Rename or delete a folder** | Copy and delete every file (slow) | One atomic operation (fast) |
| **Security** | Azure RBAC, SAS tokens, access keys | Azure RBAC plus POSIX-style ACLs per folder and file |
| **Endpoint and driver** | `blob.core.windows.net`, `wasbs://` | `dfs.core.windows.net`, `abfss://` |
| **SFTP and NFS v3** | Not available | Supported |
| **Best for** | Application files, media, backups, static websites | Data lakes and big-data analytics |

### When Blob storage is the right choice

A photo-sharing app stores each uploaded photo as a blob and shows it through its URL. Nothing scans folders, so a flat namespace is perfect.

> **Rule of thumb**
>
> File storage for an application: **Blob**. A data lake for analytics: **ADLS Gen2**.

## 9. Create a Blob storage account and compare

1. Create another storage account with the same subscription, and select the **existing** resource group `rg-session02-lab`.
2. **Name:** `stsession02blob` plus your initials. Use the same region, **Standard** and **LRS**.
3. On the **Advanced** tab, leave **Enable hierarchical namespace** unticked. Notice that **Enable SFTP** is greyed out: "SFTP can only be enabled for hierarchical namespace accounts".
4. Select **Review + create** (check **Hierarchical namespace: Disabled**), then **Create**.
5. Open **Containers**, create a container called `landing`, and open it.
6. Select **Add Directory**. Read the message: "A virtual directory does not actually exist in Azure until you paste or upload blobs into it". Create `abc` and leave it empty.
7. Upload the same test file as in section 3.

### The result

![ADLS Gen2 keeps the empty folder; Blob loses it](slides/slide-50.jpg)

- **ADLS Gen2:** the empty folder `abc` is still listed, because it is a real directory.
- **Blob:** only the file is listed. The empty virtual directory has disappeared, because a virtual directory only exists while a file name starts with it.

The Blob account's overview also shows **blob soft delete** and **container soft delete** enabled for 7 days, so you can restore files or containers deleted by mistake.

## 10. Azure Data Factory

### In one sentence

**Azure Data Factory (ADF)** is Azure's cloud service for ETL and data integration: it moves data between systems and orchestrates the steps of a data pipeline.

### What it does

- **Moves data:** copies data from more than 90 kinds of sources (databases, SaaS apps, files, APIs) into ADLS Gen2 or a warehouse.
- **Orchestrates:** runs Databricks notebooks, SQL and functions in the right order, on a schedule, with retries and alerts.
- **Low-code:** you drag activities onto a canvas and configure them. Behind the scenes everything is saved as JSON, which works with Git and CI/CD.

### ETL tools in the market

| Type | Examples |
| --- | --- |
| Traditional ETL (on-premises) | Microsoft SSIS, Informatica, IBM DataStage |
| Code-based orchestration | Apache Airflow (Python) |
| Azure-native | **Azure Data Factory** |

In an Azure project, ADF is the natural choice: it is serverless (nothing to install or patch), you pay only when pipelines run, and it integrates with ADLS Gen2, Databricks, Synapse and Key Vault. It can even run existing SSIS packages.

### Plug and play

Plug in a source connection, plug in a destination, configure a **Copy activity** and run it. Add a **trigger** to run it on a schedule and use the **Monitor** hub to watch runs and failures.

### Storage vs Data Factory

- A **storage account** is **where** data lives: 1, 10, 100 or millions of files.
- A **Data Factory** is **how and when** data moves. One factory usually holds many pipelines for a project and environment.

### Who creates these resources?

In many companies, **platform, infrastructure or cloud teams** create subscriptions, resource groups, storage accounts and data factories, often with infrastructure as code. **Data engineers** then build linked services, datasets, pipelines, triggers and transformations inside them. You still learn to create resources, because small companies need it, it helps with troubleshooting, and interviewers ask about it.

## 11. Create an Azure Data Factory

### Step by step

1. In the portal search bar, type **data factory** and select **Data factories**.
2. Select **+ Create**.
3. **Resource group:** `rg-session02-lab`.
4. **Name:** `adf-session02-lab` plus your initials (globally unique).
5. **Region:** the same region as your storage accounts. **Version:** **V2**.
6. On the **Git configuration** tab, tick **Configure Git later**.
7. Select **Review + create**, then **Create**.
8. When the deployment is complete, select **Go to resource**.

![Create Data Factory: Basics](slides/slide-60.jpg)

- All factories are **Data factory (V2)**. V1 is legacy.
- Names with suffixes such as `-dev`, `-uat` and `-prod` make the environment obvious.
- The portal may recommend **Fabric Data Factory**, the Data Factory experience in Microsoft Fabric. Azure Data Factory is fully supported and widely used, and the concepts are the same.

### If something goes wrong

- **The name is already taken:** add your initials or a number.
- **The deployment fails with InternalServerError:** this is usually a temporary problem on Azure's side. Open **Operation details** to read the real message, then select **Redeploy** or create the factory again. A failed deployment does not leave a half-built resource that you pay for.
- **The region is not available:** choose another region close to you.

> **Watch out**
>
> **Delete resource group** permanently deletes the group and everything inside it; you must type its name to confirm. Do **not** delete `rg-session02-lab` now: you use it in Session 3.

## 12. Tour of ADF Studio

The Azure portal is where you **manage** the Data Factory resource (access, networking, deletion). **ADF Studio** is where you **build** pipelines. Open your factory and select **Launch studio**; it opens at adf.azure.com.

### The five hubs

| Hub | What you do there |
| --- | --- |
| **Home** | Start pages and shortcuts |
| **Author** | Build pipelines, datasets, data flows and templates |
| **Monitor** | See pipeline runs, failures and retries |
| **Manage** | Linked services, integration runtimes, triggers and Git settings |
| **Learning** | Tutorials and samples |

### Inside a pipeline

![Pipeline canvas and activities](slides/slide-68.jpg)

- The **Activities** pane groups the steps you can add. **Copy data** (under *Move and transform*) is the most used.
- A Copy activity has tabs for **General**, **Source** (where to read), **Sink** (where to write), **Mapping** (column mapping) and **Settings**.
- Activities can be chained: a green arrow means **on success**. There are also on failure, on completion and on skip paths for error handling.
- The **General** tab sets the **timeout** (12 hours by default) and **retries**. Use 2 or 3 retries for sources that sometimes fail.
- A dataset can have **parameters** (for example a folder name), so one dataset can read many folders. **Wildcards** such as `*.csv` select many files at once.
- **Debug** test-runs a pipeline without publishing; **Publish** saves changes to the live factory; **Add trigger** schedules it.

### ADF building blocks

| Component | Meaning | Analogy |
| --- | --- | --- |
| **Pipeline** | A workflow: the ETL job | The recipe |
| **Activity** | One step (Copy, Lookup, ForEach, Notebook) | A step in the recipe |
| **Dataset** | **What** data: a file, folder or table | The ingredient |
| **Linked service** | **Where and how** to connect: connection information | The shop address and key |
| **Integration runtime** | The compute that runs activities (Azure or self-hosted) | The kitchen |
| **Trigger** | **When** a pipeline runs: schedule, tumbling window or event | The timer |

## 13. Dev, UAT and Prod environments

![Dev, UAT and Prod environments](slides/slide-73.jpg)

- Each project has **Dev** (engineers build and test), **UAT** (User Acceptance Testing: business users validate) and **Prod** (live, scheduled runs).
- Each environment has **its own** Data Factory and storage.
- Changes move from Dev to UAT to Prod through **Git and CI/CD** (for example Azure DevOps or GitHub Actions). Nobody edits production by hand.

## Wrap-up and next steps

### Key takeaways

- A **storage account** holds Blob, ADLS Gen2, Files, Queues and Tables. Its kind is StorageV2.
- **ADLS Gen2 = StorageV2 + hierarchical namespace**: real folders, ACLs and faster analytics.
- **Tenant > subscription (billing) > resource group (folder) > resource.** The region is where a resource physically lives.
- **OLTP** serves applications, **OLAP** serves analytics, and **ETL** jobs connect them. Building them is the data engineer's job.
- **Architecture:** sources > Data Factory > ADLS Gen2 > Databricks > BI, analysts and AI.
- **ADF:** pipelines, activities, datasets, linked services, integration runtimes and triggers, built in ADF Studio.

### Prepare before Session 3

Session 3 is **ADF Implementation**: the ADF end-to-end flow, linked services, datasets and the Copy activity. You will create your first cloud database (Azure SQL Database) and your first pipeline, and copy data from Azure SQL Database into ADLS Gen2 as CSV and as JSON.

- **Keep** `rg-session02-lab`, your ADLS Gen2 account (with the `landing` container) and your Data Factory. The Session 3 pipeline writes into them.
- Confirm you can open **ADF Studio** from your Data Factory.
- Post any blockers in the group so they can be fixed before Session 3.

## Practice: quiz and assignment

### Quiz

1. Which single setting turns a StorageV2 account into ADLS Gen2?
   - A. Premium performance
   - B. Enable hierarchical namespace
   - C. Geo-redundant storage
   - D. Hot access tier
2. What is a subscription in Azure?
   - A. A folder for related resources
   - B. The billing and access boundary for resources
   - C. A geographic area with datacenters
   - D. A type of storage account
3. Why does an empty folder disappear in Blob storage but not in ADLS Gen2?
4. An ATM withdrawal and a dashboard of withdrawals per city over five years: which is OLTP and which is OLAP?
5. What do E, T and L stand for in ETL?
6. An account shows *Primary: South India, Secondary: Central India*. Which redundancy option was chosen?
   - A. LRS
   - B. ZRS
   - C. GRS or RA-GRS
   - D. Premium
7. What happens to the resources when you delete a resource group?
8. In Azure Data Factory, where does the connection information for a source database live?
   - A. Pipeline
   - B. Dataset
   - C. Linked service
   - D. Trigger

<details>
<summary>Show the answers</summary>

1. B. Enable hierarchical namespace. It can be enabled later on an existing account, but it cannot be turned off once it is on.
2. B. The subscription is the billing and access boundary. Every resource is billed to exactly one subscription.
3. Blob storage has a flat namespace: a virtual folder is only part of a file name and exists only while a file uses it. ADLS Gen2 directories are real objects.
4. The ATM withdrawal is OLTP (a small, fast transaction). The five-year dashboard is OLAP (historical analysis).
5. Extract, Transform, Load.
6. C. Geo-redundant storage copies data to the paired region. The live demo used RA-GRS.
7. Every resource inside the resource group is permanently deleted.
8. C. A linked service holds the connection information. A dataset points at the data itself.

</details>

### Assignment

1. Create the ADLS Gen2 account, the Blob storage account and the Data Factory from this guide in your own subscription, with your own names.
2. In ADLS Gen2, create the containers `landing` and `curated`, and a folder path such as `landing/sales/2026/`.
3. Upload a small CSV file, then rename an ADLS Gen2 folder and notice that it is instant.
4. Repeat the empty-folder test in both accounts and take one screenshot of each result.
5. Open ADF Studio and find the **Author**, **Monitor** and **Manage** hubs, and the **Linked services** page.
6. Set a budget alert on your subscription.
7. Write 4 to 5 sentences explaining the difference between OLTP and OLAP to a friend, with an example from daily life.

**What to submit:** Your two screenshots and your short explanation. Share them the way your instructor asks (for example in the course group), and hide your subscription ID. Never include passwords, access keys, connection strings or card details.

## Key terms

| Term | Meaning |
| --- | --- |
| Storage account | A Microsoft-managed container for Azure storage services (Blob, ADLS Gen2, Files, Queues, Tables) |
| StorageV2 | General-purpose v2, the standard storage account kind |
| Blob | Binary large object: any file stored in Azure Blob storage |
| ADLS Gen2 | Azure Data Lake Storage Gen2: a StorageV2 account with hierarchical namespace enabled |
| Hierarchical namespace | Real directories, atomic rename and folder-level ACLs |
| Container | The top-level folder in a storage account (a file system in ADLS Gen2) |
| LRS / ZRS / GRS / RA-GRS | Redundancy options: local, zone, geo, and read-access geo |
| Subscription | The billing and access boundary for Azure resources |
| Resource group | A logical container for related resources |
| Region / paired region | A geographic area of datacenters / its partner region for geo-redundant copies |
| OLTP | Online Transaction Processing: databases that run applications |
| OLAP | Online Analytical Processing: data warehouses for analysis |
| ETL / ELT | Extract, Transform, Load / Extract, Load, Transform |
| Data platform | The storage, processing and pipelines a data engineer builds |
| Azure Data Factory (ADF) | Azure's cloud service for ETL, data integration and orchestration |
| Pipeline / activity | An ADF workflow / one step in it |
| Dataset / linked service | What data to use / how to connect to it |
| Integration runtime | The compute that runs ADF activities |
| Trigger | When a pipeline runs: schedule, tumbling window or event |
| Dev / UAT / Prod | Development, User Acceptance Testing and Production environments |

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)
