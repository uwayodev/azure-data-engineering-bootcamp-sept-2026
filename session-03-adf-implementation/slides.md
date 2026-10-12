# Session 3: ADF Implementation: Slides

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)

Each slide as an image, with its text underneath. See [study-guide.md](study-guide.md) for the full explanation, steps and practice. You can also [download the slides as PDF](slides.pdf).

---

## Slide 1: Azure Data Factory Implementation

![Slide 1](slides/slide-01.jpg)

- Session 03 – Live  |  Student Notes
- Linked services • Datasets • Copy activity • Azure SQL DB → ADLS Gen2 (CSV & JSON)

---

## Slide 2: Introduction: why this session matters

![Slide 2](slides/slide-02.jpg)

- Every data platform starts with one job: getting data IN (ingestion)
- Azure Data Factory (ADF) is Azure’s main tool for that job
- Today we build a real ingestion pipeline end-to-end, with zero code
- Source: Azure SQL Database (OLTP)  →  Target: ADLS Gen2 data lake
- Skills you learn today are used in almost every Azure Data Engineer interview and project

---

## Slide 3: Recap of Session 02

![Slide 3](slides/slide-03.jpg)

- Storage account: Blob, ADLS Gen2, File shares, Tables
- Blob vs ADLS Gen2: hierarchical namespace, ACLs, big-data analytics
- Subscription → Resource group → Resource;  Region = data-centre location
- OLTP (transactions) vs OLAP (analytics) and why we copy data between them
- End-to-end architecture: Sources → ADF → ADLS → Databricks → BI / AI
- We created our first Azure Data Factory resource

---

## Slide 4: Learning objectives

![Slide 4](slides/slide-04.jpg)

- Explain the ADF end-to-end flow and its 5 building blocks
- Create an Azure SQL Database with the AdventureWorksLT sample data
- Configure networking / firewall so tools can connect to the database
- Create linked services for Azure SQL Database and ADLS Gen2
- Create source and sink datasets (table, CSV, JSON)
- Build, debug, monitor and publish a pipeline with a Copy activity
- Validate the result: row counts, files, formats

---

## Slide 5: Session agenda

![Slide 5](slides/slide-05.jpg)

| # | Topic | Type |
| --- | --- | --- |
| 1 | Recap whiteboard: storage, environments, OLTP/OLAP, architecture | Theory |
| 2 | What is Azure Data Factory? ADF Studio tour | Theory + demo |
| 3 | ADF end-to-end flow: linked service, dataset, activity, pipeline, IR | Theory |
| 4 | Create the first cloud database: Azure SQL Database | Hands-on |
| 5 | Lab 1 – Azure SQL DB → ADLS Gen2 as CSV | Hands-on |
| 6 | Monitor, validate and publish | Hands-on |
| 7 | Lab 2 – Azure SQL DB → ADLS Gen2 as JSON | Hands-on |
| 8 | Troubleshooting, best practices, Q&A, homework | Discussion |

---

## Slide 6: Before we start: prerequisites

![Slide 6](slides/slide-06.jpg)

| Item | Example name used in these notes | Created in |
| --- | --- | --- |
| Azure subscription | Free trial / Pay-as-you-go / Visual Studio | Session 01 |
| Resource group | rg-adf-training | Session 02 |
| ADLS Gen2 storage account + container | stdemoadls  /  landing | Session 02 |
| Azure Data Factory (V2) | adf-demo-dev | Session 02 / today |
| Azure SQL logical server | sql-demo-server | Today |
| Azure SQL Database (sample data) | sqldb-demo (AdventureWorksLT) | Today |

> **Tip:** All names in these notes are examples. Storage, ADF and SQL server names must be globally unique – add your initials or a number.

---

## Slide 7: Part 1: Recap on the whiteboard

![Slide 7](slides/slide-07.jpg)

- Storage • Environments • OLTP vs OLAP • Architecture

---

## Slide 8: Recap: one storage account, four services

![Slide 8](slides/slide-08.jpg)

- An Azure Storage Account is a container for four data services: Blob containers (objects/files), ADLS Gen2 (Blob with hierarchical namespace for analytics), File shares (SMB/NFS network drives) and Tables (simple NoSQL key-value). Think of Blob/ADLS as a cloud drive designed for data engineering.

---

## Slide 9: Every project has Dev, UAT and Prod

![Slide 9](slides/slide-09.jpg)

- Real projects never build directly in production. Typically there are at least three environments: Dev (developers build and test), UAT/Test (business users validate with realistic data) and Prod (the live system).

---

## Slide 10: OLTP vs OLAP: and ETL in the middle

![Slide 10](slides/slide-10.jpg)

- OLTP (Online Transaction Processing) systems store the day-to-day data of applications - orders, payments, users - and are optimised for many small, fast operations. OLAP (Online Analytical Processing) systems such as a data warehouse or lakehouse store history and are optimised for large analytical queries.

---

## Slide 11: End-to-end Data Engineering project

![Slide 11](slides/slide-11.jpg)

- A typical Azure data platform: many sources (SQL/Oracle databases, SaaS applications, files, APIs, on-premises servers) → Azure Data Factory ingests the data → ADLS Gen2 stores the raw data (data lake) → Azure Databricks (or Synapse/Fabric) cleans and transforms it → the curated data is served to BI dashboards, a serving database or data warehouse, and AI/ML. Today we build the first box: ADF ingestion from Azure SQL Database into ADLS Gen2.

---

## Slide 12: Blob Storage vs ADLS Gen2

![Slide 12](slides/slide-12.jpg)

- Blob = Binary Large Object. Blob storage stores any file (images, video, PDFs, backups) in a flat namespace - 'folders' are only prefixes in the file name.

---

## Slide 13: Part 2: Azure Data Factory

![Slide 13](slides/slide-13.jpg)

- What it is • Who creates it • Create the factory • ADF Studio tour

---

## Slide 14: What is Azure Data Factory?

![Slide 14](slides/slide-14.jpg)

- Azure Data Factory (ADF) is Azure's cloud ETL and data-integration service. It lets you build data pipelines (ETL/ELT jobs) that move and transform data between more than 90 supported data stores - without writing code.

---

## Slide 15: Who creates the Azure resources?

![Slide 15](slides/slide-15.jpg)

- In many organisations the Platform / Cloud / Infrastructure team creates the Azure resources (resource groups, storage accounts, Data Factories, databases) - often using Infrastructure-as-Code such as Bicep or Terraform. Data Engineers then build inside those resources: pipelines, datasets, notebooks, data models.

---

## Slide 16: Create the Data Factory: find the service

![Slide 16](slides/slide-16.jpg)

- Portal search: type “data factories”
- Select Data factories → + Create
- Recent resources list shows what you created in earlier sessions

---

## Slide 17: Create Data Factory: Basics tab

![Slide 17](slides/slide-17.jpg)

- Subscription and resource group: rg-adf-training
- Name: globally unique, e.g. adf-demo-dev
- Region: same as your other resources
- Version: V2 (always)

---

## Slide 18: Common error: name already taken

![Slide 18](slides/slide-18.jpg)

- “The Data Factory name is already taken”
- Names are global – another customer may own it
- Fix: add initials or a number, e.g. adf-demo-dev-jw01

> **Tip:** Globally unique names: Data Factory, storage account, SQL server, Key Vault, web apps.

---

## Slide 19: Git configuration: choose “Configure Git later”

![Slide 19](slides/slide-19.jpg)

- ADF can save its code in Azure DevOps or GitHub
- Git = version control: history, review, team work
- For today: tick Configure Git later
- We use Live mode and click Publish to save

---

## Slide 20: Review + create: validation

![Slide 20](slides/slide-20.jpg)

- Validation checks required fields
- “Validation failed” = a field on a tab is missing / invalid
- Fix the tab marked with a red dot, then Create

---

## Slide 21: The factory is ready: Overview page

![Slide 21](slides/slide-21.jpg)

- Resource group, region, subscription
- Type: Data factory (V2)
- Launch studio opens ADF Studio (the authoring UI)

---

## Slide 22: ADF Studio: Home

![Slide 22](slides/slide-22.jpg)

- Home: quick starts – Ingest, Orchestrate, Transform, Configure SSIS
- Banner: Set up code repository (Git)
- Learning Center: tutorials and videos

---

## Slide 23: ADF Studio: the hubs

![Slide 23](slides/slide-23.jpg)

- Author (pencil): pipelines, datasets, data flows
- Monitor (gauge): pipeline runs, trigger runs, IR status
- Manage (toolbox): linked services, integration runtimes, Git, triggers
- Learning Center: guided tutorials

---

## Slide 24: Part 3: The ADF end-to-end flow

![Slide 24](slides/slide-24.jpg)

- Linked service • Dataset • Activity • Pipeline • Integration runtime

---

## Slide 25: Analogy: delivering an online order

![Slide 25](slides/slide-25.jpg)

- Think of an online shopping delivery. The warehouse is the source data store.

---

## Slide 26: The 5 building blocks of ADF

![Slide 26](slides/slide-26.jpg)

- 1. Linked service - the connection information to a data store or compute: server/URL, database, authentication (username/password, account key, managed identity, service principal).

---

## Slide 27: Map the analogy to ADF

![Slide 27](slides/slide-27.jpg)

| Delivery story | ADF component | In today’s lab |
| --- | --- | --- |
| Warehouse | Source data store | Azure SQL Database (sqldb-demo) |
| Agent ID card + order ID | Source linked service | SQL_DB_LS |
| Bag with the parcel | Source dataset | Customer_SQL_DB_DS (SalesLT.Customer) |
| The ride | Copy activity | Customer table to ADLS CSV |
| Gift wrap at the door | Sink dataset | ADLS_CSV_DS (DelimitedText) |
| Customer address + door code | Sink linked service | ADLS_LS (account key) |
| Customer home | Sink data store | ADLS Gen2 container landing |
| Whole journey | Pipeline | Ingest_customer_table_to_adls |
| Fuel / power | Integration runtime | AutoResolveIntegrationRuntime |

---

## Slide 28: ADF end-to-end flow: SQL table to CSV

![Slide 28](slides/slide-28.jpg)

- Read the flow from left to right: Azure SQL DB → linked service (server, database, user, password) → dataset (the Customer table) → Copy activity → dataset (CSV file in ADLS) → linked service (storage account URL + key) → ADLS Gen2. The source side always has linked service + dataset, the sink side always has dataset + linked service.

---

## Slide 29: Integration runtime: the compute behind ADF

![Slide 29](slides/slide-29.jpg)

- Azure IR (AutoResolveIntegrationRuntime): managed, serverless, for cloud sources
- Self-hosted IR: installed on a VM/PC to reach on-premises or private networks
- Azure-SSIS IR: lift-and-shift of existing SSIS packages
- Copy performance unit: DIU (Data Integration Unit) – more DIUs = more power, more cost
- Today we use the default AutoResolveIntegrationRuntime

---

## Slide 30: Part 4: Create the first cloud database

![Slide 30](slides/slide-30.jpg)

- Azure SQL Database with AdventureWorksLT sample data

---

## Slide 31: Azure SQL: logical server and databases

![Slide 31](slides/slide-31.jpg)

- Azure SQL Database is a fully managed relational database (PaaS) - Microsoft handles patching, backups and high availability. Every database lives on a logical SQL server.

---

## Slide 32: Find Azure SQL → SQL databases → Create

![Slide 32](slides/slide-32.jpg)

- Search “sql database” → Azure SQL
- SQL databases → + Create → SQL database
- Free offer option exists – good for learning

---

## Slide 33: Basics: resource group and database name

![Slide 33](slides/slide-33.jpg)

- Resource group: rg-adf-training
- Database name: sqldb-demo
- Server: Create new (next slide)
- Elastic pool: No;  Workload: Development

---

## Slide 34: Create the logical SQL server

![Slide 34](slides/slide-34.jpg)

- Server name: globally unique (sql-demo-server)
- Location: same region as the database
- Authentication: SQL authentication (for the lab)
- Admin login + strong password – write them down!

> **Tip:** Password rules: 8–128 characters, 3 of: uppercase, lowercase, number, symbol. Cannot contain the login name.

---

## Slide 35: Compute + storage: keep it cheap

![Slide 35](slides/slide-35.jpg)

- Service tier: General Purpose
- Compute tier: Serverless (auto-pause, billed per second)
- Hardware: Standard-series, 1 vCore, small max size
- Backup redundancy: Locally-redundant (cheapest)

---

## Slide 36: Networking: public endpoint and firewall

![Slide 36](slides/slide-36.jpg)

- Connectivity method: Public endpoint
- Allow Azure services and resources to access this server: Yes
- Add current client IP address: Yes
- Production: prefer Private endpoint

---

## Slide 37: Additional settings: load sample data

![Slide 37](slides/slide-37.jpg)

- Use existing data: Sample
- Creates the AdventureWorksLT database
- Tables in schema SalesLT: Customer, Product, Address, SalesOrderHeader …
- Gives us real data to copy – no scripts needed

---

## Slide 38: Review + create

![Slide 38](slides/slide-38.jpg)

- Check the estimated cost (storage + compute per vCore-second)
- Click Create
- Deployment takes a few minutes

---

## Slide 39: Deployment failed?  Read the error!

![Slide 39](slides/slide-39.jpg)

- Status: Conflict → Error details
- “Invalid value given for parameter Password”
- Fix the password (complexity rules) and Redeploy / create again

> **Tip:** Always open Error details / Operation details – the real reason is written there.

---

## Slide 40: Deployment complete: database Overview

![Slide 40](slides/slide-40.jpg)

- Your deployment is complete → Go to resource
- Server name, status Online, pricing tier, auto-pause delay
- Connection strings available from the Overview page

---

## Slide 41: Query editor: sign in

![Slide 41](slides/slide-41.jpg)

- Database → Query editor (preview)
- Login with SQL authentication (sqladmin + password)
- Error: “Client with IP address … is not allowed”
- Click Allowlist IP … to add a firewall rule

---

## Slide 42: Explore the sample tables

![Slide 42](slides/slide-42.jpg)

- Tables: SalesLT.Customer, Product, Address, SalesOrderHeader …
- SELECT * FROM [SalesLT].[Customer]
- Query succeeded – the database is ready

---

## Slide 43: Part 5: Lab 1: Azure SQL DB → ADLS as CSV

![Slide 43](slides/slide-43.jpg)

- Pipeline → Linked services → Datasets → Copy activity → Debug → Publish

---

## Slide 44: Lab 1 plan

![Slide 44](slides/slide-44.jpg)

- Objects we create in Lab 1: 1. Pipeline: Ingest_customer_table_to_adls 2.

---

## Slide 45: Step 1: Create a pipeline

![Slide 45](slides/slide-45.jpg)

- Author hub → + (Factory Resources) → Pipeline → Pipeline
- A blank canvas opens with the Activities pane
- Publish all shows the number of unsaved changes

---

## Slide 46: Step 1: Name the pipeline

![Slide 46](slides/slide-46.jpg)

- Properties pane → Name
- Ingest_customer_table_to_adls
- Use a clear verb + object + target convention

> **Tip:** Naming: letters, numbers and underscores are the safest choice for every ADF object name.

---

## Slide 47: Step 2: Manage hub → Linked services

![Slide 47](slides/slide-47.jpg)

- Manage (toolbox) → Connections → Linked services
- + New → Data store gallery
- 90+ connectors: Azure, AWS, databases, files, SaaS

---

## Slide 48: Step 2a: Linked service for Azure SQL Database

![Slide 48](slides/slide-48.jpg)

- Search “sql” → Azure SQL Database → Continue
- Name: SQL_DB_LS
- Connect via integration runtime: AutoResolveIntegrationRuntime
- Version 2.0 (Recommended)

---

## Slide 49: Step 2a: Server, database and credentials

![Slide 49](slides/slide-49.jpg)

- Account selection: From Azure subscription
- Server name: sql-demo-server;  Database: sqldb-demo
- Authentication: SQL authentication
- User name: sqladmin + Password
- Production: store the password in Azure Key Vault

---

## Slide 50: Step 2a: Test connection → Create

![Slide 50](slides/slide-50.jpg)

- Test connection → Connection successful
- Create
- SQL_DB_LS appears in the list

> **Tip:** Test fails? Check: firewall “Allow Azure services” = Yes, correct server/database, correct password.

---

## Slide 51: Step 2b: Linked service for ADLS Gen2

![Slide 51](slides/slide-51.jpg)

- + New → search “data lake” → Azure Data Lake Storage Gen2
- Name: ADLS_LS
- Authentication type: Account key
- Account selection: From Azure subscription

---

## Slide 52: Step 2b: Select storage account and test

![Slide 52](slides/slide-52.jpg)

- Storage account name: stdemoadls
- Test connection → successful → Create
- Now we have 2 linked services: ADLS_LS and SQL_DB_LS

---

## Slide 53: Step 3: Create the source dataset

![Slide 53](slides/slide-53.jpg)

- Author → + → Dataset
- Search “sql” → Azure SQL Database → Continue

---

## Slide 54: Dataset naming rule: a common error

![Slide 54](slides/slide-54.jpg)

- Name “Customer-SQL_DB_DS” is rejected
- “Dataset name should only contain letters, numbers or underscores”
- Use Customer_SQL_DB_DS

---

## Slide 55: Step 3: Set properties: linked service + table

![Slide 55](slides/slide-55.jpg)

- Name: Customer_SQL_DB_DS
- Linked service: SQL_DB_LS
- Table name: SalesLT.Customer
- Import schema: From connection/store → OK

---

## Slide 56: Source dataset created

![Slide 56](slides/slide-56.jpg)

- Connection tab: linked service + table
- Schema tab: imported columns
- Preview data to check the rows

---

## Slide 57: Step 4: Create the sink dataset (CSV)

![Slide 57](slides/slide-57.jpg)

- + Dataset → Azure Data Lake Storage Gen2
- Select format: DelimitedText (CSV)
- Name: ADLS_CSV_DS;  Linked service: ADLS_LS

---

## Slide 58: Step 4: File path and header

![Slide 58](slides/slide-58.jpg)

- File path: container / directory / file name
- Browse → container landing
- First row as header: ticked
- Import schema: None (file does not exist yet)

---

## Slide 59: Sink dataset settings

![Slide 59](slides/slide-59.jpg)

- Compression: none
- Column delimiter: comma (,)
- Row delimiter: default (\r\n or \n)
- Encoding: UTF-8

---

## Slide 60: Step 5: Add the Copy data activity

![Slide 60](slides/slide-60.jpg)

- Open the pipeline tab
- Activities → Move and transform → Copy data
- Drag onto the canvas;  rename on the General tab

---

## Slide 61: Step 5: Source and Sink tabs

![Slide 61](slides/slide-61.jpg)

- Source dataset: Customer_SQL_DB_DS
- Sink dataset: ADLS_CSV_DS
- Red “1” markers disappear when required fields are filled

---

## Slide 62: Source options worth knowing

![Slide 62](slides/slide-62.jpg)

- Use query: Table / Query / Stored procedure
- Query lets you filter columns and rows (SELECT … WHERE …)
- Query timeout, isolation level, partition option

---

## Slide 63: Step 6: Debug the pipeline

![Slide 63](slides/slide-63.jpg)

- Debug = test run without publishing
- Output tab shows pipeline run ID and status
- Queued → In progress → Succeeded

---

## Slide 64: Debug succeeded

![Slide 64](slides/slide-64.jpg)

- Pipeline status: Succeeded
- Activity Copy data1: Succeeded
- Glasses icon = Details of the copy

---

## Slide 65: Copy details: what happened?

![Slide 65](slides/slide-65.jpg)

- Rows read: 847  →  Rows written: 847
- Files written: 1
- Copy duration, throughput, Used DIUs: 4
- Source region vs sink region

---

## Slide 66: Validate in the source: row count

![Slide 66](slides/slide-66.jpg)

- SELECT COUNT(*) FROM [SalesLT].[Customer]
- Result: 847
- Matches rows written ✓

---

## Slide 67: Validate in ADLS: the file

![Slide 67](slides/slide-67.jpg)

- Storage account → Containers → landing
- New file: SalesLT.Customer.txt
- File name comes from the source table name

---

## Slide 68: Open the CSV file

![Slide 68](slides/slide-68.jpg)

- Blob properties: URL, size, access tier
- Edit tab shows the content
- Header row + comma-separated values ✓

---

## Slide 69: How do we prove the copy worked?

![Slide 69](slides/slide-69.jpg)

- Validation checklist for every ingestion job: 1. Pipeline and activity status = Succeeded. 2.

---

## Slide 70: Step 7: Publish all

![Slide 70](slides/slide-70.jpg)

- Live mode: nothing is saved until you Publish
- Publish all → review pending changes → Publish
- Pipelines + datasets (+ linked services) are saved

> **Tip:** Closing the browser before publishing = losing your work in Live mode.

---

## Slide 71: Publishing completed

![Slide 71](slides/slide-71.jpg)

- “Publishing completed – Successfully published”
- Publish all counter returns to 0
- Rename activities to meaningful names (e.g. Customer table to ADLS CSV)

---

## Slide 72: Live mode vs Git mode

![Slide 72](slides/slide-72.jpg)

- Live mode: no repository; changes are drafts until Publish all writes them to the ADF service. Fine for learning.

---

## Slide 73: What Git mode looks like

![Slide 73](slides/slide-73.jpg)

- Branch selector: main branch, adf_publish branch
- New branch, Create pull request, Switch to live mode
- Toolbar: Save all + Publish

---

## Slide 74: Part 6: Lab 2: Azure SQL DB → ADLS as JSON

![Slide 74](slides/slide-74.jpg)

- Reuse what you built – only the format changes

---

## Slide 75: Lab 2: how many new objects?

![Slide 75](slides/slide-75.jpg)

- Requirement: copy the SalesLT.Product table into ADLS Gen2 as a JSON file. • Linked services: 0 new - reuse SQL_DB_LS and ADLS_LS (same database, same storage). • Datasets: 2 new - Product_SQL_DS (table SalesLT.Product) and ADLS_JSON_DS (JSON format in landing). • Activities: 1 Copy data activity. • Pipelines: 1 new pipeline (Ingest_product_table_to_adls). Key idea: linked services are reusable connections; datasets are specific to a table/file and format.

---

## Slide 76: Source dataset for Product

![Slide 76](slides/slide-76.jpg)

- + Dataset → Azure SQL Database
- Name: Product_SQL_DS
- Linked service: SQL_DB_LS (reused)
- Table: SalesLT.Product

---

## Slide 77: Sink dataset in JSON format

![Slide 77](slides/slide-77.jpg)

- + Dataset → ADLS Gen2 → format JSON
- Name: ADLS_JSON_DS
- Linked service: ADLS_LS (reused)

---

## Slide 78: JSON dataset settings

![Slide 78](slides/slide-78.jpg)

- File path: landing
- Compression: none;  Encoding: UTF-8
- Factory resources now show 5 datasets

---

## Slide 79: Pipeline + Copy activity for Product

![Slide 79](slides/slide-79.jpg)

- New pipeline: Ingest_product_table_to_adls
- Copy data activity: Product sql table to ADLS JSON

---

## Slide 80: Source = Product_SQL_DS, Sink = ADLS_JSON_DS

![Slide 80](slides/slide-80.jpg)

- Source tab: Product_SQL_DS, Use query = Table
- Sink tab: ADLS_JSON_DS

---

## Slide 81: Debug and monitor

![Slide 81](slides/slide-81.jpg)

- Debug → Queued → In progress → Succeeded
- Duration ~30 seconds

---

## Slide 82: Copy details for the JSON load

![Slide 82](slides/slide-82.jpg)

- Rows read 295 = rows written 295
- Data read ~715 KB, data written ~999 KB
- JSON is bigger: every row repeats the column names

---

## Slide 83: Validate the JSON output

![Slide 83](slides/slide-83.jpg)

- SELECT COUNT(*) FROM [SalesLT].[Product] → 295 ✓
- landing/SalesLT.Product.json
- One JSON object per line (JSON Lines / set of objects)

---

## Slide 84: Publish all again

![Slide 84](slides/slide-84.jpg)

- Pending changes: new pipeline + datasets (New), edited pipeline (Edited)
- Publish → work is saved

---

## Slide 85: Part 7: Wrap-up

![Slide 85](slides/slide-85.jpg)

- Troubleshooting • Best practices • Q&A • Homework

---

## Slide 86: Troubleshooting cheat-sheet

![Slide 86](slides/slide-86.jpg)

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Name already taken | Global uniqueness (ADF, storage, SQL server) | Add initials / numbers |
| SQL deployment failed: invalid Password | Complexity rules not met | Use 3 of 4 character types, 8+ chars |
| Client IP not allowed (Query editor) | Firewall rule missing / IP changed | Allowlist IP or add client IP in Networking |
| Linked service test fails (SQL) | “Allow Azure services” = No, wrong password | Enable it; re-enter credentials |
| Linked service test fails (ADLS) | Wrong account, key rotated, network rules | Re-select account; check firewall/key |
| Dataset name rejected | Hyphens or spaces in name | Use letters, numbers, underscores |
| Work disappeared after closing browser | Not published (Live mode) | Always Publish all (or use Git) |
| Rows written ≠ source count | Query filter, wrong table, partial failure | Check source query and activity output |

---

## Slide 87: Best practices from day one

![Slide 87](slides/slide-87.jpg)

- Naming convention for every object (LS / DS / PL prefixes or suffixes)
- Reuse linked services; one linked service per data store, per environment
- Never hard-code secrets: Azure Key Vault or managed identity
- Keep source, ADF and storage in the same region
- Test connection, Debug, then Publish; validate row counts every time
- Git integration + CI/CD for real projects (Dev → UAT → Prod)
- Use Serverless / free tiers and delete resources you do not need

---

## Slide 88: Frequently asked questions

![Slide 88](slides/slide-88.jpg)

| Question | Short answer |
| --- | --- |
| Is ADF ETL or ELT? | Both. Copy = E + L; transform in ADF data flows or in Databricks/SQL (ELT). |
| Linked service vs dataset? | Linked service = how to connect; dataset = which data + format. |
| Why Debug and not Trigger now? | Debug tests the draft; Trigger runs the published version. |
| Can one pipeline copy many tables? | Yes – parameters + Lookup + ForEach (coming in later sessions). |
| On-prem source? | Install a Self-hosted Integration Runtime. |
| How is ADF billed? | Per activity run, data-movement DIU-hours, and data flow cluster time. |
| ADF vs Fabric Data Factory? | Same concepts; Fabric is SaaS inside Microsoft Fabric. ADF skills transfer. |
| CSV, JSON or Parquet in the lake? | Parquet/Delta for analytics; CSV/JSON for simple exchange. |

---

## Slide 89: Homework / practice

![Slide 89](slides/slide-89.jpg)

- Rebuild Lab 1 and Lab 2 from scratch without the recording
- Copy SalesLT.Address to ADLS as CSV into a folder named address/
- Use a Query in the source: only customers whose CompanyName starts with ‘A’
- Set the sink file name to customer.csv instead of the default name
- Bonus: try the Parquet format and compare the file size with CSV and JSON
- Write 5 lines explaining linked service, dataset, activity, pipeline, IR

---

## Slide 90: Next session preview

![Slide 90](slides/slide-90.jpg)

- Parameterised linked services and datasets (one dataset, many tables)
- Lookup, ForEach and control-flow activities
- Triggers: schedule and event-based
- Monitoring and alerts

---

## Slide 91: Thank you

![Slide 91](slides/slide-91.jpg)

- Questions?  ·  Publish your work  ·  Practise Lab 1 and Lab 2

---

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)
