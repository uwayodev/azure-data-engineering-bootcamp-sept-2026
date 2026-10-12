# Session 3: ADF Implementation (Study guide)

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)

**What you will learn:** You will understand how Azure Data Factory moves data from end to end, and what its five building blocks do: linked services, datasets, activities, pipelines and integration runtimes. You will create your first cloud database (Azure SQL Database with sample data), connect it to Data Factory, and build two pipelines with a Copy activity: one copies a table into ADLS Gen2 as CSV, the other as JSON. You will also learn to debug, monitor, validate and publish a pipeline.

**Before you start:** The resource group, the ADLS Gen2 account (with its `landing` container) and the Data Factory from Session 2, and Google Chrome on a laptop or desktop.

**Slides:** [slides.md](slides.md) · [Download the slides as PDF](slides.pdf)

> **Names used in this guide**
>
> This guide reuses the names from Session 2 and adds the database. SQL server names must be **unique across all of Azure**, so add your initials or a number. The screenshots use other example names (`rg-adf-training`, `adf-demo-dev`, `stdemoadls`, `sql-demo-server`, `sqldb-demo`): use your own.
>
> | Item | Name in this guide | Rule |
> | --- | --- | --- |
> | Resource group | `rg-session02-lab` | From Session 2 |
> | ADLS Gen2 account and container | `stsession02adls` / `landing` | From Session 2 |
> | Data Factory | `adf-session02-lab` | From Session 2 |
> | Logical SQL server | `sql-session03-lab` | Globally unique; lowercase letters, numbers and hyphens |
> | SQL database | `sqldb-session03` | Unique on its server |
> | SQL admin login | `sqladmin` | Cannot be `admin`, `sa` or `root` |
> | Linked services | `SQL_DB_LS`, `ADLS_LS` | Letters, numbers and underscores |
> | Datasets | `Customer_SQL_DB_DS`, `ADLS_CSV_DS`, `Product_SQL_DS`, `JSON_ADLS_DS` | Letters, numbers and underscores (no hyphens) |
> | Pipelines | `Ingest_customer_table_to_adls`, `Ingest_product_data` | Letters, numbers and underscores |

## Recap of Session 2

- A **storage account** holds Blob, ADLS Gen2, Files, Queues and Tables. **ADLS Gen2** is a StorageV2 account with **hierarchical namespace** on: real folders, ACLs and faster analytics.
- **Subscription** (who pays) > **resource group** (logical folder) > **resource**. The **region** is where a resource physically lives.
- **OLTP** databases run applications; **OLAP** systems serve analytics; **ETL** jobs copy data between them.
- **Architecture:** sources > Azure Data Factory > ADLS Gen2 > Databricks > BI, analysts and AI.
- You created an **Azure Data Factory** and toured **ADF Studio**.

How Session 2 connects to today: the ADLS Gen2 `landing` container becomes the **destination** of today's pipelines, and the Data Factory you created is where you build them. Azure SQL Database plays the role of the **OLTP** source system.

![End-to-end data engineering project](slides/slide-11.jpg)

Today you build the first box of this architecture: **ingestion with Azure Data Factory**.

## 1. Azure Data Factory in one page

### In one sentence

**Azure Data Factory (ADF)** is Azure's serverless, low-code service for building data pipelines: it copies data between more than 90 kinds of data stores and orchestrates the steps of an ETL or ELT job.

![What is Azure Data Factory](slides/slide-14.jpg)

- **Zero coding:** you drag activities onto a canvas and fill in their properties. Behind the scenes everything is saved as JSON.
- **Serverless:** there is nothing to install or patch. You pay per activity run and for the data movement, not for idle servers.
- **Similar tools:** Apache Airflow, Informatica, SSIS and IBM DataStage. The same pipeline engine also exists in **Azure Synapse Analytics** and **Microsoft Fabric** (Data Factory in Fabric), so the skills transfer.

### Who creates the resources?

![Who creates the Azure resources](slides/slide-15.jpg)

In many companies the **platform or cloud team** creates the resource groups, storage accounts, databases and data factories, often with infrastructure as code. **Data engineers** then build inside them: linked services, datasets and pipelines. There is usually **one Data Factory per environment** (for example `adf-sales-dev`, `adf-sales-uat`, `adf-sales-prod`), and changes move from Dev to UAT to Prod with Git and CI/CD.

### Reminder: creating the factory

You created your factory in Session 2. If you need a new one, these are the key settings:

1. Search for **Data factories** and select **+ Create**.
2. **Resource group:** `rg-session02-lab`. **Name:** globally unique. **Region:** the same as your storage. **Version:** **V2**.
3. On the **Git configuration** tab, tick **Configure Git later**.
4. Select **Review + create**, then **Create**, then **Go to resource** and **Launch studio**.

![Name already taken](slides/slide-18.jpg)

- **The Data Factory name is already taken:** names are global because they become part of a public address. Add your initials or a number.
- **Validation failed:** the tab with the problem shows a red dot. Fix that field and try again.

### The ADF Studio hubs

![ADF Studio hubs](slides/slide-23.jpg)

| Hub | What you do there | Memory hook |
| --- | --- | --- |
| **Author** (pencil) | Pipelines, datasets, data flows | Logic |
| **Monitor** (gauge) | Pipeline runs, trigger runs, failures | Results |
| **Manage** (toolbox) | Linked services, integration runtimes, Git, triggers | Connections |
| **Home** and **Learning Center** | Shortcuts, wizards and tutorials | Start here |

The **Azure portal** is where you manage the factory resource (access, tags, networking). **ADF Studio** is where you build pipelines.

**Think about it:** Where would you go to find out why last night's pipeline failed?

## 2. The ADF end-to-end flow

### An easy analogy: delivering an online order

![Analogy: delivering an online order](slides/slide-25.jpg)

A parcel leaves a **warehouse**. The delivery agent needs an **ID card and the order number** to collect it. The parcel travels in a **bag**, is moved by a **ride**, and may be **gift-wrapped** again at the door. The customer's **address and door code** let the agent deliver it to the **customer's home**. The whole journey is planned as one trip, and the ride needs **fuel**.

| Delivery story | ADF component | In today's Lab 1 |
| --- | --- | --- |
| Warehouse | Source data store | Azure SQL Database |
| Agent ID card and order number | Source **linked service** | `SQL_DB_LS` |
| Bag with the parcel | Source **dataset** | `Customer_SQL_DB_DS` (table `SalesLT.Customer`) |
| The ride | **Copy activity** | Customer table to ADLS CSV |
| Gift wrap at the door | Sink **dataset** | `ADLS_CSV_DS` (CSV format) |
| Customer address and door code | Sink **linked service** | `ADLS_LS` |
| Customer home | Sink data store | ADLS Gen2, container `landing` |
| Whole journey | **Pipeline** | `Ingest_customer_table_to_adls` |
| Fuel | **Integration runtime** | `AutoResolveIntegrationRuntime` |

### The five building blocks

![The 5 building blocks of ADF](slides/slide-26.jpg)

1. **Linked service:** **how to connect**. The server or URL, the database and the authentication (user name and password, account key, managed identity). Think of it as a connection string.
2. **Dataset:** **what data**. A specific table, file or folder inside a linked service, plus its format (CSV, JSON, Parquet, and so on).
3. **Activity:** **one step of work**. Data movement (Copy), transformation (data flow, Databricks notebook, stored procedure) or control (Lookup, ForEach, If Condition, Wait).
4. **Pipeline:** **the job**. A logical group of activities that you run, schedule and monitor.
5. **Integration runtime (IR):** **the compute** that runs the activities.

### The flow from left to right

![ADF end-to-end flow](slides/slide-28.jpg)

- The **source side** always has a linked service and a dataset; the **sink side** (destination) always has a dataset and a linked service.
- The **Copy activity** reads the source dataset and writes the sink dataset.
- The **order you create things in ADF Studio:** 1) linked services, 2) datasets, 3) pipeline and activity, 4) debug, 5) publish.

### Integration runtime types

| Type | Use it when | Example |
| --- | --- | --- |
| **Azure IR** (`AutoResolveIntegrationRuntime`) | Source and sink are cloud services reachable from Azure | Today's labs |
| **Self-hosted IR** | Data is on-premises or in a private network | A SQL Server in a company datacenter |
| **Azure-SSIS IR** | You need to run existing SSIS packages | Lift and shift of old ETL |

Copy performance is measured in **DIUs (Data Integration Units)**. More DIUs mean more power and more cost. You will see *Used DIUs: 4* in today's run details.

> **Interview question**
>
> "How do you copy data from an on-premises SQL Server to ADLS Gen2?" Install a **self-hosted integration runtime** inside the company network, create a linked service that uses it, then use a Copy activity.

**Think about it:** Which building block stores the password? Which one knows that the file is CSV and not JSON?

## 3. Create your first cloud database: Azure SQL Database

### Logical server and database

![Azure SQL: logical server and databases](slides/slide-31.jpg)

**Azure SQL Database** is a fully managed relational database (PaaS): Microsoft handles patching, backups and high availability.

- Every database lives on a **logical SQL server**. It is not a virtual machine; it is a management container that holds the **admin login**, the **firewall rules**, the Microsoft Entra admin and the region.
- One logical server can host **many databases**, each with its own size and cost.
- To connect you need four things: **server name** (`<name>.database.windows.net`), **database name**, **login** and **password** (or Microsoft Entra ID).

### Step by step

1. In the portal search bar, type **SQL database** and open **Azure SQL** > **SQL databases**. Select **+ Create** > **SQL database**. (If you see **SQL database (Free offer)**, it gives a monthly free amount and is a good choice for learning.)
2. **Resource group:** `rg-session02-lab`.
3. **Database name:** `sqldb-session03`.
4. **Server:** select **Create new** and fill in the server form (next section).
5. **Want to use SQL elastic pool?** **No**. **Workload environment:** **Development**.
6. **Compute + storage:** select **Configure database** and choose a low-cost option (see below).
7. **Backup storage redundancy:** **Locally-redundant backup storage**.
8. On the **Networking** tab, set the firewall rules (see below).
9. On the **Additional settings** tab, set **Use existing data** to **Sample**.
10. Select **Review + create**, check the estimated cost, then **Create**. The deployment takes a few minutes.

![Create SQL Database: Basics](slides/slide-33.jpg)

### Create the logical SQL server

![Create the logical SQL server](slides/slide-34.jpg)

- **Server name:** `sql-session03-lab` plus your initials. It becomes `sql-session03-lab.database.windows.net`.
- **Location:** the same region as your other resources.
- **Authentication method:** for this lab, **Use SQL authentication** (or **Use both SQL and Microsoft Entra authentication**). Microsoft recommends Microsoft Entra-only authentication in production.
- **Server admin login:** `sqladmin`. **Password:** 8 to 128 characters, with characters from three of these groups: uppercase, lowercase, numbers and symbols. It cannot contain the login name.

> **Watch out**
>
> Write down the server name, login and password now. You need them for the Query editor and for the ADF linked service. Never post them in the group or commit them to Git.

### Compute + storage: keep it cheap

![Compute and storage](slides/slide-35.jpg)

| Setting | Lab choice | Why |
| --- | --- | --- |
| **Service tier** | General Purpose | Budget-friendly for most workloads |
| **Compute tier** | **Serverless** | Billed per second; **auto-pauses** when idle (for example after 1 hour) |
| **Max vCores / data max size** | The smallest values | The sample database is tiny |
| **Backup redundancy** | Locally-redundant | The cheapest option |

**Provisioned** compute is billed every hour, even when nobody uses it. The cost summary is shown in the currency of your billing account.

### Networking: let ADF and your laptop in

![Networking and firewall](slides/slide-36.jpg)

By default nothing can connect to a new SQL server. On the **Networking** tab:

- **Connectivity method:** **Public endpoint**.
- **Allow Azure services and resources to access this server:** **Yes**. Without it, the ADF linked-service test fails.
- **Add current client IP address:** **Yes**. This lets you use the Query editor from your laptop.

In production, companies usually prefer **private endpoints** instead of public access.

### Additional settings: sample data

![Load sample data](slides/slide-37.jpg)

**Use existing data: Sample** creates **AdventureWorksLT**, a small database for a fictitious bicycle company. Its tables are in the `SalesLT` schema: `Customer` (847 rows), `Product` (295 rows), `Address`, `SalesOrderHeader` and more. It gives you real data to copy without writing any scripts. If the portal says the sample changes your compute settings, select **OK**.

### If something goes wrong

![Deployment failed](slides/slide-39.jpg)

- **Your deployment failed, status Conflict:** open **Error details**. In the live demo the message was *Invalid value given for parameter Password*: the password did not meet the rules. Create the database again with a valid password.
- **The server name is already taken:** add your initials or a number.

> **Tip**
>
> Always read **Error details** or **Operation details** before guessing. The real reason is written there.

When you see **Your deployment is complete**, select **Go to resource**. The **Overview** page shows the server name, the status (**Online**), the pricing tier and the auto-pause delay.

### Query the database

![Query editor sign-in and firewall error](slides/slide-41.jpg)

1. In your database, open **Query editor (preview)**.
2. Sign in with **SQL server authentication**: `sqladmin` and your password.
3. If you see *Client with IP address ... is not allowed to access the server*, select the **Allowlist IP ...** link, or open the server > **Networking** > **Firewall rules** > **Add your client IPv4 address**. Your home or mobile IP address can change, so this error can come back another day.
4. Expand **Tables** and run:

```sql
SELECT * FROM [SalesLT].[Customer];

SELECT COUNT(*) FROM [SalesLT].[Customer];   -- 847
SELECT COUNT(*) FROM [SalesLT].[Product];    -- 295
```

![Explore the sample tables](slides/slide-42.jpg)

Note the two counts: you compare them with the pipeline results later.

**Think about it:** Why does ADF need *Allow Azure services* while your laptop needs *Add current client IP address*?

## 4. Lab 1: copy a table to ADLS Gen2 as CSV

### The plan

You create six objects, in this order:

| # | Object | Type | Details |
| --- | --- | --- | --- |
| 1 | `Ingest_customer_table_to_adls` | Pipeline | The job |
| 2 | `SQL_DB_LS` | Linked service | Azure SQL Database |
| 3 | `ADLS_LS` | Linked service | Azure Data Lake Storage Gen2 |
| 4 | `Customer_SQL_DB_DS` | Dataset | Table `SalesLT.Customer` |
| 5 | `ADLS_CSV_DS` | Dataset | DelimitedText (CSV) in `landing` |
| 6 | Customer table to ADLS CSV | Copy activity | Source 4, sink 5 |

Then you **debug**, **validate** and **publish**.

### Step 1: create the pipeline

1. In ADF Studio, open the **Author** hub.
2. Next to **Factory Resources**, select **+** > **Pipeline** > **Pipeline**.
3. Open the **Properties** pane (top right) and set **Name** to `Ingest_customer_table_to_adls`.

![Create a pipeline](slides/slide-45.jpg)

- The **Activities** pane on the left lists the steps you can add; the canvas is in the middle.
- The **Publish all** button shows how many changes are not saved yet.

### Step 2: create the linked services

Linked services are created in the **Manage** hub: **Manage** > **Connections** > **Linked services** > **+ New**.

**2a. Azure SQL Database**

1. Search for **sql**, select **Azure SQL Database** and **Continue**.
2. **Name:** `SQL_DB_LS`. **Connect via integration runtime:** `AutoResolveIntegrationRuntime`. **Version:** **2.0 (Recommended)**.
3. **Account selection method:** **From Azure subscription**. Select your subscription, **Server name** and **Database name**.
4. **Authentication type:** **SQL authentication**. **User name:** `sqladmin`. **Password:** your password.
5. Select **Test connection**. When you see **Connection successful**, select **Create**.

![Linked service for Azure SQL Database](slides/slide-49.jpg)

- The **Password | Azure Key Vault** switch: in real projects the secret is stored in **Azure Key Vault** and the linked service only refers to it. **Managed identity** authentication is even better, because there is no password at all.

**2b. ADLS Gen2**

1. Select **+ New** again, search for **data lake**, select **Azure Data Lake Storage Gen2** (not Azure Blob Storage) and **Continue**.
2. **Name:** `ADLS_LS`. **Authentication type:** **Account key**.
3. **Account selection method:** **From Azure subscription**. Select your **Storage account name** (`stsession02adls`).
4. **Test connection**, then **Create**.

![Linked service for ADLS Gen2](slides/slide-52.jpg)

You now have two linked services. Each one is a **reusable connection**: any number of datasets can use it.

### Step 3: create the source dataset

1. Back in **Author**, select **+** > **Dataset**.
2. Search for **sql**, select **Azure SQL Database** and **Continue**.
3. **Name:** `Customer_SQL_DB_DS`. **Linked service:** `SQL_DB_LS`.
4. **Table name:** `SalesLT.Customer`. **Import schema:** **From connection/store**. Select **OK**.

![Set properties: linked service and table](slides/slide-55.jpg)

- **Import schema** reads the column names and types, so the Copy activity can map the columns automatically.
- Use **Preview data** on the dataset to check that it points at the right rows.

![Dataset naming rule](slides/slide-54.jpg)

> **Watch out**
>
> *Dataset name should only contain letters, numbers or underscores.* `Customer-SQL_DB_DS` is rejected because of the hyphen. Use underscores in every ADF object name.

### Step 4: create the sink dataset (CSV)

1. Select **+** > **Dataset** > **Azure Data Lake Storage Gen2** > **Continue**.
2. **Select format:** **DelimitedText**. This is CSV in ADF.
3. **Name:** `ADLS_CSV_DS`. **Linked service:** `ADLS_LS`.
4. **File path:** browse and select the `landing` container. Leave **Directory** and **File name** empty.
5. Tick **First row as header**. **Import schema:** **None** (the file does not exist yet). Select **OK**.

![File path and header](slides/slide-58.jpg)

| Setting | Value | Meaning |
| --- | --- | --- |
| **File path** | `landing` / *(directory)* / *(file name)* | Container, folder, file. An empty file name lets ADF name the file after the source table |
| **First row as header** | Ticked | Column names are written on the first line |
| **Compression type** | None | Could be gzip, snappy and others |
| **Column delimiter** | Comma (,) | Could be semicolon, tab or pipe |
| **Encoding** | UTF-8 | The standard text encoding |

Other formats you can choose: Avro, Binary, Excel, Iceberg, **JSON**, ORC, **Parquet** and XML.

### Step 5: add the Copy activity

1. Open the pipeline tab.
2. In **Activities**, search for **copy** (or open **Move and transform**) and drag **Copy data** onto the canvas.
3. **General** tab: **Name** = `Customer table to ADLS CSV`.
4. **Source** tab: **Source dataset** = `Customer_SQL_DB_DS`.
5. **Sink** tab: **Sink dataset** = `ADLS_CSV_DS`.

![Source and Sink tabs](slides/slide-61.jpg)

- A small red number on a tab means a required field is still empty.
- **Use query** on the Source tab can be **Table** (the whole table), **Query** (your own `SELECT ... WHERE ...`) or **Stored procedure**. A query with a date filter is the basis of **incremental loading**, which you will meet later.

### Step 6: debug and monitor

1. Select **Debug** on the pipeline toolbar. Debug runs your **unpublished** pipeline as a test.
2. Watch the **Output** tab: the status goes from **Queued** to **In progress** to **Succeeded**.
3. Hover over the activity row and select the **Details** icon (glasses).

![Copy details](slides/slide-65.jpg)

| Field | Lab 1 result | What to check |
| --- | --- | --- |
| **Rows read** | 847 | Equals the source `COUNT(*)` |
| **Rows written** | 847 | Equals rows read |
| **Files written** | 1 | One file was created |
| **Copy duration** | About 17 seconds | Normal for a small table |
| **Used DIUs** | 4 | The compute power used |
| **Regions** | Source and sink regions | Different regions add latency and may add cost |

> **Tip**
>
> In the live demo the database was in East US and the storage in South India, so the data crossed regions. In your own lab, keep the database, the factory and the storage in the **same region**.

### Validate the result

![Validate in ADLS: the file](slides/slide-67.jpg)

1. In the Query editor, run `SELECT COUNT(*) FROM [SalesLT].[Customer];` and confirm **847**.
2. In the storage account, open **Containers** > `landing`. A new file named after the table, `SalesLT.Customer.txt`, is there.
3. Open the file and select **Edit**: the first line is the header (`CustomerID,NameStyle,Title,FirstName,...`) and every other line is one customer.

![How do we prove the copy worked](slides/slide-69.jpg)

**Validation checklist (use it for every pipeline):**

1. The pipeline and the activity show **Succeeded**.
2. **Rows read = rows written = source `COUNT(*)`**.
3. The file exists in the expected folder, with the expected format.
4. Open the file: the header, delimiter and values look right.

### Step 7: publish

1. Select **Publish all**.
2. Review the **Pending changes** (pipeline and datasets) and select **Publish**.
3. Wait for **Publishing completed**.

![Publish all](slides/slide-70.jpg)

> **Watch out**
>
> Your factory is in **Live mode** (no Git). Pipelines and datasets are only drafts in your browser until you select **Publish all**. Close the tab before publishing and your work is lost. Delete accidental objects (for example an unused `DelimitedText1` dataset) before you publish.

### Live mode vs Git mode

![Live mode vs Git mode](slides/slide-72.jpg)

|   | Live mode (today) | Git mode (real projects) |
| --- | --- | --- |
| **Where changes are stored** | Drafts in the browser until **Publish all** | Committed to Azure DevOps or GitHub with **Save all** |
| **Teamwork** | Hard: everyone edits the same factory | Feature branches and pull requests |
| **History and rollback** | None | Full version history |
| **Deployment to UAT and Prod** | Manual | **Publish** creates ARM templates in the `adf_publish` branch, and CI/CD deploys them |

You chose **Configure Git later** when you created the factory. You can connect a repository at any time in **Manage** > **Git configuration**.

**Think about it:** What is the difference between **Save all** in Git mode and **Publish** in Live mode?

## 5. Lab 2: copy a table to ADLS Gen2 as JSON

### How many new objects?

![Lab 2: how many new objects](slides/slide-75.jpg)

The requirement: copy `SalesLT.Product` into the `landing` container as **JSON**. Before you start, decide what you need:

| Object | New? | Why |
| --- | --- | --- |
| Linked services | **0** | Same database and same storage: reuse `SQL_DB_LS` and `ADLS_LS` |
| Datasets | **2** | A dataset is specific to one table or file and one format |
| Activities | **1** | One Copy data activity |
| Pipelines | **1** | A new pipeline for products |

> **Key idea**
>
> Linked services are **connections** and are reused. Datasets are **specific** to a table or file and its format.

### Step by step

Try it on your own first, then check these steps.

1. **Source dataset:** **+** > **Dataset** > **Azure SQL Database**. **Name:** `Product_SQL_DS`. **Linked service:** `SQL_DB_LS`. **Table:** `SalesLT.Product`. **Import schema:** **From connection/store**.
2. **Sink dataset:** **+** > **Dataset** > **Azure Data Lake Storage Gen2** > format **JSON**. **Name:** `JSON_ADLS_DS`. **Linked service:** `ADLS_LS`. **File path:** `landing`.
3. **Pipeline:** **+** > **Pipeline**. **Name:** `Ingest_product_data`.
4. **Copy activity:** drag **Copy data** onto the canvas. **Name:** `Product sql table to ADLS JSON`. **Source:** `Product_SQL_DS`. **Sink:** `JSON_ADLS_DS`.
5. **Debug**, and wait for **Succeeded**.
6. **Validate:** `SELECT COUNT(*) FROM [SalesLT].[Product];` returns **295**, and `landing` contains `SalesLT.Product.json`.
7. **Publish all**.

![Sink dataset in JSON format](slides/slide-77.jpg)

> **Tip**
>
> Check spelling in names. The live demo used `Ingets_product_data` by mistake. Names appear in monitoring and alerts, so a typo stays with you.

### Why is the JSON file bigger?

![Copy details for the JSON load](slides/slide-82.jpg)

The run read **295 rows (about 715 KB)** and wrote **295 rows (about 999 KB)**. The output is bigger because every JSON record repeats every column name:

```json
{"ProductID":680,"Name":"HL Road Frame - Black, 58","ProductNumber":"FR-R92B-58","Color":"Black","StandardCost":1059.31, ...}
{"ProductID":706,"Name":"HL Road Frame - Red, 58","ProductNumber":"FR-R92R-58","Color":"Red","StandardCost":1059.31, ...}
```

A CSV file writes the column names only once, in the header. By default ADF writes **one JSON object per line** (JSON Lines, called *set of objects*); you can change the sink to an *array of objects*.

| Format | Strength | Typical use |
| --- | --- | --- |
| **CSV** | Simple, opens in Excel | Data exchange with people and older systems |
| **JSON** | Nested structures, self-describing | APIs, web and NoSQL data |
| **Parquet** | Columnar, compressed, fast to query | **Data lakes and analytics** (Session 4) |

![Validate the JSON output](slides/slide-83.jpg)

**Think about it:** Why do data lakes prefer Parquet over CSV and JSON?

## 6. Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| *Name is already taken* | Data Factory, storage and SQL server names are global | Add your initials or a number |
| SQL deployment failed: *Invalid value given for parameter Password* | The password does not meet the rules | 8+ characters from 3 of the 4 groups; no login name inside |
| *Client with IP address ... is not allowed* (Query editor) | No firewall rule for your IP, or your IP changed | **Allowlist IP**, or add your client IP in **Networking** |
| SQL linked service test fails | *Allow Azure services* is off, or the wrong password | Turn it on in **Networking**; enter the credentials again |
| ADLS linked service test fails | Wrong account, rotated key or storage firewall | Select the account again; check the key and **Networking** |
| *Dataset name should only contain letters, numbers or underscores* | A hyphen or space in the name | Use underscores |
| Work disappeared after closing the browser | Not published (Live mode) | Always **Publish all**, or use Git mode |
| Rows written differ from the source count | A query filter, the wrong table, or a failed run | Check the Source tab and the activity output |

## 7. Best practices from day one

- Use a **naming convention** for every object and stick to it. This guide uses suffixes (`SQL_DB_LS`, `ADLS_CSV_DS`); prefixes (`LS_SQL_Sales`, `DS_ADLS_Customer_CSV`, `PL_Ingest_Customer`) are also common.
- **Reuse linked services:** one per data store, per environment.
- **Never hard-code secrets:** use Azure Key Vault or managed identity.
- Keep the source, the factory and the storage in the **same region**.
- **Test connection**, then **Debug**, then **Publish**, and validate the row counts every time.
- Use **Git integration and CI/CD** in real projects (Dev > UAT > Prod).
- Use **Serverless** or free tiers, and delete what you do not need.

## Wrap-up and next steps

### Key takeaways

- **Linked service = how to connect. Dataset = what data and which format. Activity = one step. Pipeline = the job. Integration runtime = the compute.**
- Build in this order: linked services, datasets, pipeline and activity, debug, publish.
- **Azure SQL Database** lives on a logical server. Open the **firewall** for Azure services (ADF) and for your client IP (Query editor).
- The **Copy activity** reads a source dataset and writes a sink dataset. Changing the sink format (CSV to JSON) only needs a new dataset, not new linked services.
- **Validate** every load: status, rows read = rows written = source count, the file and its content.
- In **Live mode**, nothing is saved until you **Publish all**.

### Prepare before Session 4

Session 4 is **ADF Pipelines**: parameters, queries, Parquet, dynamic folders, and copying many tables with one pipeline.

- **Keep** `rg-session02-lab`, your storage account, your Data Factory and the SQL database. Session 4 builds on today's linked services and datasets.
- The serverless database **auto-pauses** when idle, so it costs very little between sessions. The first query after a pause can take a minute while it resumes.
- Make sure both of today's pipelines are **published** and show **Succeeded** in a debug run.
- Post any blockers in the group so they can be fixed before Session 4.

## Practice: quiz and assignment

### Quiz

1. Which ADF component holds the server name and the password for a database?
   - A. Dataset
   - B. Linked service
   - C. Pipeline
   - D. Trigger
2. Which ADF component knows that the data is a CSV file in the `landing` container?
   - A. Linked service
   - B. Integration runtime
   - C. Dataset
   - D. Activity
3. Name the five building blocks of an ADF solution.
4. A company wants to copy data from a SQL Server in its own datacenter to ADLS Gen2. Which integration runtime does it need?
   - A. Azure integration runtime
   - B. Self-hosted integration runtime
   - C. Azure-SSIS integration runtime
   - D. No integration runtime is needed
5. The linked-service test to Azure SQL Database fails, but the password is correct. Which server setting should you check first?
6. Lab 2 copies a different table from the same database to the same storage account, as JSON. How many new linked services do you need, and why?
7. What is the difference between **Debug** and **Publish all** in Live mode?
8. The Copy activity read 847 rows. What else must you check to prove the copy was complete?
9. Why is the JSON output larger than the data read from the database?
10. Why is `Customer-SQL_DB_DS` rejected as a dataset name?

<details>
<summary>Show the answers</summary>

1. B. A linked service holds the connection information: server, database and authentication.
2. C. A dataset points at specific data (a table, file or folder) and its format.
3. Linked service, dataset, activity, pipeline and integration runtime.
4. B. A self-hosted integration runtime is installed inside the private network and connects out to ADF.
5. **Allow Azure services and resources to access this server** in the server's **Networking** settings. Without it, ADF's Azure integration runtime cannot reach the database.
6. None. Linked services are reusable connections, and the database and the storage account are the same. You only need new datasets (and a pipeline and an activity).
7. **Debug** test-runs the current draft without saving it. **Publish all** saves your pipelines and datasets to the live factory. Without publishing, the work is lost when you close the browser.
8. Rows written must equal rows read, and both must equal `SELECT COUNT(*)` on the source table. Then check that the file exists in the right folder and that its content looks right.
9. Every JSON record repeats all the column names, while a CSV file writes them only once, in the header.
10. Dataset names can only contain letters, numbers and underscores. The hyphen is not allowed.

</details>

### Assignment

1. Rebuild Lab 1 and Lab 2 in your own subscription, with your own names, without watching the recording.
2. Copy `SalesLT.Address` to ADLS Gen2 as CSV into a folder named `address` (set the **Directory** in the sink dataset).
3. On the Source tab, choose **Query** and copy only the customers whose company name starts with A:

   ```sql
   SELECT * FROM [SalesLT].[Customer] WHERE CompanyName LIKE 'A%';
   ```

   Compare **Rows read** with the result of the same query in the Query editor.
4. Set the sink **File name** to `customer.csv` instead of the default name.
5. Bonus: create a **Parquet** sink dataset, copy the Customer table again, and compare the file size with the CSV and JSON files.
6. Write 5 lines explaining linked service, dataset, activity, pipeline and integration runtime in your own words.

**What to submit:** A screenshot of each successful debug run (showing **Succeeded** and the rows read and written), the screenshot of your `landing` container with the new files, and your 5-line explanation. Share them the way your instructor asks (for example in the course group), and hide your subscription ID and email address. Never include passwords, access keys, connection strings or card details.

## Key terms

| Term | Meaning |
| --- | --- |
| Azure Data Factory (ADF) | Azure's serverless service for data integration, ETL and orchestration |
| ADF Studio | The web interface (adf.azure.com) where you build pipelines |
| Linked service | Connection information to a data store or compute service |
| Dataset | A named reference to data in a linked service: a table, file or folder, and its format |
| Activity | One step in a pipeline (Copy, Lookup, ForEach, Notebook) |
| Copy activity | The activity that reads a source dataset and writes a sink dataset |
| Pipeline | A logical group of activities: the job you run, schedule and monitor |
| Integration runtime (IR) | The compute that runs activities: Azure, self-hosted or Azure-SSIS |
| AutoResolveIntegrationRuntime | The default Azure IR, managed by Microsoft |
| DIU | Data Integration Unit: the measure of Copy activity compute power |
| Source / sink | Where the data is read from / where it is written to |
| DelimitedText | ADF's name for CSV and other delimited text formats |
| JSON Lines | One JSON object per line (ADF's *set of objects* pattern) |
| Debug | A test run of the unpublished pipeline |
| Publish all | Saves pipelines and datasets to the live factory (Live mode) |
| Live mode / Git mode | Working without / with a Git repository connected to the factory |
| Azure SQL Database | A fully managed relational database service (PaaS) |
| Logical SQL server | The container for databases, logins and firewall rules |
| Serverless compute | Compute that scales automatically, is billed per second and can auto-pause |
| Firewall rule | Allows an IP address or Azure services to connect to the SQL server |
| AdventureWorksLT | Microsoft's small sample database (schema `SalesLT`) |
| Azure Key Vault | Azure's service for storing secrets such as passwords and keys |

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)
