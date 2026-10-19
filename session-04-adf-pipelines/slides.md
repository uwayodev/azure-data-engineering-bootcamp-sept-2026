# Session 4: ADF Pipelines: Slides

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)

Each slide as an image, with its text underneath. See [study-guide.md](study-guide.md) for the full explanation, steps and practice. You can also [download the slides as PDF](slides.pdf).

---

## Slide 1: ADF Pipelines in Practice

![Slide 1](slides/slide-01.jpg)

- Session 04 – Live  |  Student Notes
- Parquet • Source queries & joins • Many tables in one pipeline • Dynamic folders • Dataset parameters

---

## Slide 2: Introduction: why this session matters

![Slide 2](slides/slide-02.jpg)

- Real projects copy tens or hundreds of tables, not one
- The data lake must be organised: the right format, the right folder
- Copy only the data you need: queries, filters, joins
- Reusable building blocks (parameters) instead of copy-paste
- These patterns appear in almost every ADF interview and project

---

## Slide 3: Recap of Session 03

![Slide 3](slides/slide-03.jpg)

- Quick recap of the vocabulary: • Linked service - how to connect (server, database, credentials, storage URL and key). • Dataset - what data: a table, or a file/folder in a given format. • Copy activity - reads the source dataset and writes the sink dataset. • Pipeline - a group of activities, the unit you run and monitor. • Integration runtime - the compute; we use AutoResolveIntegrationRuntime. • Debug runs the draft version; Publish all saves the work in Live mode.

---

## Slide 4: Learning objectives

![Slide 4](slides/slide-04.jpg)

- Explain structured, semi-structured and unstructured data
- Choose between CSV, JSON and Parquet – and explain why Parquet wins in a lake
- Use a source Query: select columns, filter rows, join tables
- Put several Copy activities in one pipeline; chain them or run them in parallel
- Use the 4 dependency conditions: success, failure, completion, skip
- Write each table into its own folder in ADLS
- Build one parameterised dataset and pass values at run time

---

## Slide 5: Session agenda

![Slide 5](slides/slide-05.jpg)

| # | Topic | Type |
| --- | --- | --- |
| 1 | Types of data and file formats: CSV, JSON, Parquet | Theory |
| 2 | Lab 1 – CustomerAddress table → ADLS as Parquet | Hands-on |
| 3 | Source queries – Lab 2 (Query → JSON) and Lab 3 (Join) | Hands-on |
| 4 | Lab 4 – Many tables in one pipeline, dependency conditions | Hands-on |
| 5 | Lab 5 – Dynamic folder creation in the lake | Hands-on |
| 6 | Lab 6 – Dataset parameters (dynamic parameterisation) | Hands-on |
| 7 | Lab 7 – Practice: ProductDescription (with a real error) | Hands-on |
| 8 | Troubleshooting, best practices, Q&A, homework | Discussion |

---

## Slide 6: Before we start: what you need

![Slide 6](slides/slide-06.jpg)

| Item | Example name used in these notes | Created in |
| --- | --- | --- |
| Resource group | rg-adf-training | Session 02 |
| ADLS Gen2 storage account + container | stdemoadls  /  landing | Session 02 |
| Azure Data Factory (V2) | adf-demo-dev | Session 02 / 03 |
| Azure SQL server + database (sample data) | sql-demo-server / sqldb-demo | Session 03 |
| Linked services | SQL_DB_LS  and  ADLS_LS | Session 03 |
| Datasets | Customer_SQL_DB_DS, ADLS_CSV_DS | Session 03 |

> **Tip:** The serverless database may be paused – the first connection can take up to a minute while it wakes up.

---

## Slide 7: Part 1: Types of data and file formats

![Slide 7](slides/slide-07.jpg)

- Structured • Semi-structured • Unstructured • CSV vs JSON vs Parquet

---

## Slide 8: Three types of data

![Slide 8](slides/slide-08.jpg)

- • Structured data has a fixed schema of rows and columns: relational tables, CSV files, Excel sheets. It is easy to query with SQL. • Semi-structured data carries its structure with it, using keys or tags, and the schema can change from record to record: JSON, XML, log files.

---

## Slide 9: CSV vs JSON vs Parquet

![Slide 9](slides/slide-09.jpg)

- CSV (DelimitedText) is plain text, one row per line, column names once in a header. It is simple and readable but has no data types and no built-in compression.

---

## Slide 10: Row storage vs columnar storage

![Slide 10](slides/slide-10.jpg)

- Row-based formats (CSV, JSON, and OLTP databases) store a complete record together. That is perfect for reading or writing one record at a time.

---

## Slide 11: Today's labs at a glance

![Slide 11](slides/slide-11.jpg)

- Pipelines and datasets we build today (example names): 1. Ingest_customer_address - SalesLT.CustomerAddress to Parquet (dataset Customer_address_sql_DS, ADLS_Parquet_DS) 2.

---

## Slide 12: Part 2: Lab 1: Table → ADLS as Parquet

![Slide 12](slides/slide-12.jpg)

- SalesLT.CustomerAddress  →  landing/SalesLT.CustomerAddress.parquet

---

## Slide 13: Step 1: New pipeline: Ingest_customer_address

![Slide 13](slides/slide-13.jpg)

- Author → + → Pipeline → Pipeline
- Properties → Name: Ingest_customer_address
- Factory Resources already show last week’s pipelines

---

## Slide 14: Step 2: Add a Copy data activity

![Slide 14](slides/slide-14.jpg)

- Activities → Move and transform → Copy data
- Name: CustomerAddress table to ADLS parquet
- Red numbers on tabs = required fields missing

---

## Slide 15: Step 3: New source dataset (Azure SQL Database)

![Slide 15](slides/slide-15.jpg)

- Source tab → + New
- Search “sql” → Azure SQL Database → Continue
- You can create datasets from the activity – no need to leave the pipeline

---

## Slide 16: Step 3: Name, linked service and table

![Slide 16](slides/slide-16.jpg)

- Name: Customer_address_sql_DS
- Linked service: SQL_DB_LS (reused)
- Import schema: From connection/store

---

## Slide 17: Step 3: Select SalesLT.CustomerAddress

![Slide 17](slides/slide-17.jpg)

- Table drop-down → SalesLT.CustomerAddress
- Connection tab: linked service + table
- Preview data to check

---

## Slide 18: Step 4: Sink: new ADLS Gen2 dataset

![Slide 18](slides/slide-18.jpg)

- Source dataset now selected; Use query = Table
- Sink tab → + New
- Search “data lake” → Azure Data Lake Storage Gen2

---

## Slide 19: Step 4: Choose the Parquet format

![Slide 19](slides/slide-19.jpg)

- Formats: Avro, Binary, DelimitedText, Excel, Iceberg, JSON, ORC, Parquet, XML
- Select Parquet → Continue

---

## Slide 20: Step 4: ADLS_Parquet_DS: linked service and path

![Slide 20](slides/slide-20.jpg)

- Name: ADLS_Parquet_DS
- Linked service: ADLS_LS (reused)
- Browse → landing container

---

## Slide 21: Step 4: File path and import schema

![Slide 21](slides/slide-21.jpg)

- File path: landing / (directory) / (file name)
- Leave directory and file name empty
- Import schema: None (no file yet) – then OK

> **Tip:** Import schema “From connection/store” needs an existing file. For a new sink, choose None.

---

## Slide 22: Step 5: Debug the pipeline

![Slide 22](slides/slide-22.jpg)

- Sink dataset: ADLS_Parquet_DS
- Debug = test run without publishing
- Output tab: Queued → In progress → Succeeded

---

## Slide 23: Debug succeeded

![Slide 23](slides/slide-23.jpg)

- Pipeline status: Succeeded
- Activity: CustomerAddress table to ADLS parquet
- Duration ≈ 28 s

---

## Slide 24: Copy details: rows and regions

![Slide 24](slides/slide-24.jpg)

- Rows read 417 = rows written 417
- Data read ≈ 22.5 KB → written ≈ 21.3 KB (Parquet)
- Source region East US → sink region South India
- Used DIUs 4, parallel copies 1

---

## Slide 25: Validate in ADLS: the Parquet file

![Slide 25](slides/slide-25.jpg)

- landing → SalesLT.CustomerAddress.parquet
- Named after the source table
- Size ≈ 20.8 KiB

---

## Slide 26: Parquet is binary: you cannot read it in Notepad

![Slide 26](slides/slide-26.jpg)

- Edit tab: “may not render correctly … unrecognized extension”
- Binary columnar content, not text
- Read it with Databricks, Synapse, Fabric, Spark, Power BI

---

## Slide 27: Part 3: Source queries and joins

![Slide 27](slides/slide-27.jpg)

- Lab 2: Query → JSON   •   Lab 3: Join two tables

---

## Slide 28: Table, Query or Stored procedure?

![Slide 28](slides/slide-28.jpg)

- The Copy activity Source tab has a 'Use query' option: • Table - copy the whole table (all rows, all columns). • Query - run your own SELECT statement: choose columns, filter rows with WHERE, join tables, rename columns. • Stored procedure - call a procedure in the database that returns rows.

---

## Slide 29: Lab 2: Pipeline Ingest_address_data

![Slide 29](slides/slide-29.jpg)

- New pipeline: Ingest_address_data
- Copy data activity
- Error: “Activity name should not have trailing space”

---

## Slide 30: Lab 2: Reuse an existing SQL dataset

![Slide 30](slides/slide-30.jpg)

- Activity name: Address table to ADLS
- Source dataset: any dataset on SQL_DB_LS
- With Use query = Query the table setting is ignored

---

## Slide 31: Lab 2: Use query: Query

![Slide 31](slides/slide-31.jpg)

- Preview data shows the dataset’s own table (Customer)
- Use query → Query
- Query timeout default: 120 minutes

---

## Slide 32: Lab 2: Write the query

![Slide 32](slides/slide-32.jpg)

- SELECT * FROM SalesLT.Address
- Preview data now shows Address rows
- Query box supports Add dynamic content

---

## Slide 33: Lab 2: Sink: ADLS_JSON_DS

![Slide 33](slides/slide-33.jpg)

- Sink dataset list: ADLS_JSON_DS, ADLS_Parquet_DS, DelimitedText1 …
- Choose ADLS_JSON_DS (JSON in landing)
- Same sink dataset works for any table

---

## Slide 34: Lab 2: Debug and copy details

![Slide 34](slides/slide-34.jpg)

- Succeeded in ~24 s
- Rows read 450 = rows written 450
- Read ≈ 60 KB → JSON written ≈ 117 KB

---

## Slide 35: Lab 2: Validate: the file is called data_<guid>

![Slide 35](slides/slide-35.jpg)

- New file: data_abf5c25f-… (not SalesLT.Address)
- With a query ADF cannot use a table name
- One JSON object per line

---

## Slide 36: Where does my file go: and what is it called?

![Slide 36](slides/slide-36.jpg)

- Summary of ADF sink naming rules: • Table source, empty file name → file named after the table (SalesLT.CustomerAddress.parquet). • Query source, empty file name → data_<guid>.<ext>. • Directory set → ADF creates the folder(s) automatically. • File name set → exactly that name; running again overwrites it. Folders are cheap and are created on the fly, but schema import on a folder that does not exist yet fails.

---

## Slide 37: Lab 3: Explore the tables to join

![Slide 37](slides/slide-37.jpg)

- Database → Query editor (preview)
- Firewall error? Allowlist your IP (see Session 03)
- SELECT * FROM SalesLT.CustomerAddress

---

## Slide 38: Joining two tables in the source query

![Slide 38](slides/slide-38.jpg)

- A JOIN combines rows from two tables that share a key. Customer and CustomerAddress share CustomerID.

---

## Slide 39: Lab 3: Pipeline Joined_data and the join query

![Slide 39](slides/slide-39.jpg)

- New pipeline: Joined_data
- select * from salesLT.customer c join salesLT.customeraddress cd on c.customerid = cd.customerid

---

## Slide 40: Lab 3: Error: duplicate column names

![Slide 40](slides/slide-40.jpg)

- Error code 9019: duplicate column names “CustomerID”
- Fix: select explicit columns
- select c.customerid, c.firstname, cd.addresstype …

> **Tip:** Never use SELECT * on a join in a pipeline – list the columns.

---

## Slide 41: Lab 3: Preview the joined result

![Slide 41](slides/slide-41.jpg)

- Columns: customerid, firstname, addresstype
- Rows like 29485 – Catherine – Main Office
- Choose a sink (CSV/Parquet) and Debug

---

## Slide 42: Part 4: Many tables in one pipeline

![Slide 42](slides/slide-42.jpg)

- Lab 4: Customer + Address + Product  •  dependency conditions  •  sequential vs parallel

---

## Slide 43: The 4 dependency conditions

![Slide 43](slides/slide-43.jpg)

- Activities are connected with dependency arrows. Each activity has four output handles: • On success (green) - the next activity runs only if this one succeeded.

---

## Slide 44: Lab 4: Pipeline Multiple_table_ingestion

![Slide 44](slides/slide-44.jpg)

- New pipeline: Multiple_table_ingestion
- First Copy activity: Customer Data

---

## Slide 45: Lab 4: Customer Data: source and sink

![Slide 45](slides/slide-45.jpg)

- Source: Customer_SQL_DB_DS (Table)
- Sink: ADLS_CSV_DS
- Reuse last week’s datasets

---

## Slide 46: Lab 4: Add a second activity and connect it

![Slide 46](slides/slide-46.jpg)

- Copy or drag a second Copy data activity
- Drag the green handle of Customer Data onto it
- Green arrow = On success

---

## Slide 47: The four handles in ADF Studio

![Slide 47](slides/slide-47.jpg)

- On skip (grey arrow)
- On success (green)
- On fail (red)

---

## Slide 48: On completion and the Address activity

![Slide 48](slides/slide-48.jpg)

- On completion (blue): runs after success or failure
- Rename the second activity: Address data

---

## Slide 49: Lab 4: Address data: query + CSV sink

![Slide 49](slides/slide-49.jpg)

- Use query: select * from saleslt.address
- Sink: ADLS_CSV_DS

---

## Slide 50: Lab 4: Third activity: Product data

![Slide 50](slides/slide-50.jpg)

- Third Copy activity: Product data
- Connected On success from Address data
- Use query: select * from saleslt.product

---

## Slide 51: Lab 4: Product data sink and the full chain

![Slide 51](slides/slide-51.jpg)

- Sink: ADLS_CSV_DS
- Customer → Address → Product, all On success

---

## Slide 52: Run the chain: one after the other

![Slide 52](slides/slide-52.jpg)

- Activities start only when the previous one succeeds
- Address 26 s → Customer 25 s → Product 28 s
- Total ≈ 80 s

---

## Slide 53: Sequential or parallel?

![Slide 53](slides/slide-53.jpg)

- If the activities are not connected, ADF starts them all at the same time - in parallel. The pipeline duration becomes roughly the duration of the slowest activity.

---

## Slide 54: Run them in parallel

![Slide 54](slides/slide-54.jpg)

- Delete the arrows → 3 independent activities
- All three start at the same time
- Queued together at the same timestamp

---

## Slide 55: Parallel result: and the files

![Slide 55](slides/slide-55.jpg)

- All three succeeded, each ~26 s, total ≈ 27 s
- Files: data_<guid> names in landing
- Problem: one flat folder – which file is which?

---

## Slide 56: Part 5: Dynamic folder creation

![Slide 56](slides/slide-56.jpg)

- Lab 5: write each table into its own folder in the lake

---

## Slide 57: Lab 5: Pipeline Customer_data_to_folder

![Slide 57](slides/slide-57.jpg)

- New pipeline: Customer_data_to_folder
- Copy data1 source query: select * from saleslt.customer

---

## Slide 58: Set a directory on the sink dataset

![Slide 58](slides/slide-58.jpg)

- Sink: ADLS_CSV_DS → Open
- File path: landing / Customer / (file name)
- ADF creates the folder automatically

> **Tip:** Editing a shared dataset changes EVERY pipeline that uses it.

---

## Slide 59: Second activity: Product data

![Slide 59](slides/slide-59.jpg)

- Product data activity, connected On success
- Sink → + New → ADLS Gen2

---

## Slide 60: New dataset: Product_Folder_CSV_DS

![Slide 60](slides/slide-60.jpg)

- Format: DelimitedText
- Name: Product_Folder_CSV_DS
- First row as header ✓

---

## Slide 61: File path landing / Product: schema import error

![Slide 61](slides/slide-61.jpg)

- File path: landing / Product
- Import schema: From connection/store → error
- “PathNotFound – the specified path does not exist”

> **Tip:** New sink folder? Set Import schema to None.

---

## Slide 62: Debug the folder pipeline

![Slide 62](slides/slide-62.jpg)

- Sink: Product_Folder_CSV_DS
- Debug → Customer data then Product data

---

## Slide 63: Validate: one folder per table

![Slide 63](slides/slide-63.jpg)

- Pipeline succeeded
- landing now has a Customer folder
- Folders sort above files

---

## Slide 64: Inside the Customer folder

![Slide 64](slides/slide-64.jpg)

- landing/Customer/data_<guid>
- CSV with header row
- Customer columns: CustomerID, NameStyle, Title, FirstName …

---

## Slide 65: Product folder: but two datasets for two folders

![Slide 65](slides/slide-65.jpg)

- landing: Customer and Product folders
- Works – but one dataset per folder
- 100 tables = 100 datasets?

---

## Slide 66: Part 6: Dataset parameters

![Slide 66](slides/slide-66.jpg)

- Lab 6: one generic dataset, folder name passed at run time

---

## Slide 67: Hard-coded vs parameterised datasets

![Slide 67](slides/slide-67.jpg)

- A parameter is a placeholder whose value is supplied when the pipeline runs. Instead of one dataset per folder, we create ONE dataset with a folderName parameter.

---

## Slide 68: How a dataset parameter works

![Slide 68](slides/slide-68.jpg)

- 1. In the dataset, Parameters tab → + New → name folderName, type String. 2.

---

## Slide 69: Lab 6: New dataset ADLS_Generic_DS

![Slide 69](slides/slide-69.jpg)

- + New dataset → ADLS Gen2
- Format: DelimitedText

---

## Slide 70: ADLS_Generic_DS: properties

![Slide 70](slides/slide-70.jpg)

- Name: ADLS_Generic_DS
- Linked service: ADLS_LS
- File path: landing / (empty) / (empty)
- Import schema: None

---

## Slide 71: Parameters tab → + New

![Slide 71](slides/slide-71.jpg)

- Dataset → Parameters tab
- + New: Name, Type, Default value

---

## Slide 72: Create the folderName parameter

![Slide 72](slides/slide-72.jpg)

- Name: folderName,  Type: String
- Back to Connection → click Directory
- “Add dynamic content [Alt+Shift+D]”

---

## Slide 73: Pipeline expression builder

![Slide 73](slides/slide-73.jpg)

- Lists parameters, functions, system variables
- Click folderName → @dataset().folderName
- OK

---

## Slide 74: The Directory is now dynamic

![Slide 74](slides/slide-74.jpg)

- Directory: @dataset().folderName
- Copy activity Sink: ADLS_Generic_DS
- Dataset properties → folderName = Value

---

## Slide 75: Pass a value from each activity

![Slide 75](slides/slide-75.jpg)

- Customer data → folderName = Customer_Folder
- Product data → folderName = Product_Folder
- Same dataset, different folders

---

## Slide 76: Debug and check the run

![Slide 76](slides/slide-76.jpg)

- Customer data → Product data
- Both succeeded (~26 s each)

---

## Slide 77: Validate: new folders from one dataset

![Slide 77](slides/slide-77.jpg)

- landing: Customer_Folder and Product_Folder
- File: landing/Customer_Folder/data_<guid>
- No new dataset needed for a new table

---

## Slide 78: Part 7: Lab 7: Practice

![Slide 78](slides/slide-78.jpg)

- Copy SalesLT.ProductDescription into its own folder – on your own

---

## Slide 79: Lab 7: Pipeline Ingest_product_desc

![Slide 79](slides/slide-79.jpg)

- New pipeline: Ingest_product_desc
- Source query: select * from productdescription

---

## Slide 80: Lab 7: Sink: ADLS_Generic_DS

![Slide 80](slides/slide-80.jpg)

- Sink: ADLS_Generic_DS
- folderName = ProductDescription
- Debug → Queued

---

## Slide 81: Lab 7: The run fails: read the error

![Slide 81](slides/slide-81.jpg)

- Error code 2200 – User configuration issue
- “Invalid object name ‘productdescription’”
- Failure happened on the Source side

> **Tip:** Invalid object name = wrong table name or missing schema.

---

## Slide 82: Lab 7: Fix and re-run

![Slide 82](slides/slide-82.jpg)

- select * from saleslt.productdescription
- Debug → Succeeded in ~27 s

---

## Slide 83: Lab 7: Validate the new folder

![Slide 83](slides/slide-83.jpg)

- landing: ProductDescription folder
- landing/ProductDescription/data_<guid> (~145 KiB)
- Publish all when done

---

## Slide 84: Part 8: Wrap-up

![Slide 84](slides/slide-84.jpg)

- Troubleshooting • Best practices • Q&A • Homework

---

## Slide 85: Troubleshooting cheat-sheet

![Slide 85](slides/slide-85.jpg)

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Activity name should not have trailing space | Space at end of the name | Remove the space |
| Duplicate column names (9019) | SELECT * on a join | List columns; rename with AS |
| Schema import failed: PathNotFound | Sink folder does not exist yet | Import schema = None |
| Invalid object name (2200) | Missing schema (SalesLT.) or typo | Use schema.table |
| Client IP not allowed (Query editor) | Firewall / IP changed | Allowlist IP on the server |
| Files named data_<guid> | Query source + no file name | Set a file name or folder |
| Other pipelines write to a new folder | Shared dataset was edited | Use a parameterised dataset |
| Parquet file looks corrupted | It is binary | Read it with Spark / Synapse / Fabric |

---

## Slide 86: Best practices from today

![Slide 86](slides/slide-86.jpg)

- Parquet (or Delta) for analytics; CSV/JSON only for exchange
- Copy only what you need: explicit columns, WHERE filters
- Never SELECT * on a join; always prefix tables with the schema
- One folder per table (and later per load date)
- Parameterise datasets – avoid one dataset per table
- Run independent copies in parallel; chain only real dependencies
- Use On failure for alerts and logging

---

## Slide 87: Frequently asked questions

![Slide 87](slides/slide-87.jpg)

| Question | Short answer |
| --- | --- |
| Parquet or Delta? | Delta = Parquet files + transaction log (ACID, time travel). Use Delta in Databricks/Fabric. |
| Why did my query file get a strange name? | Query source + no file name → data_<run id>. |
| Do I need to create folders first? | No – ADF creates them during the copy. |
| Max parallel activities? | Many; keep the source database load in mind. |
| Dataset vs pipeline parameter? | Dataset parameter makes the dataset reusable; pipeline parameter is set when the pipeline runs. |
| How do I loop over many tables? | Lookup (get the list) + ForEach (repeat a Copy) – later in the course. |
| Can I filter rows changed since yesterday? | Yes – a WHERE on ModifiedDate: incremental load. |
| Does Debug cost money? | Yes, like a normal run (activity runs + DIU time), but small. |

---

## Slide 88: Quick quiz

![Slide 88](slides/slide-88.jpg)

| Question | Answer |
| --- | --- |
| Which format is columnar and binary? | Parquet |
| Which source option lets you join two tables? | Use query = Query |
| Which dependency runs only if the previous activity failed? | On failure (red) |
| Three unconnected Copy activities run … | In parallel |
| Expression for a dataset parameter called folderName? | @dataset().folderName |
| Why did “select * from productdescription” fail? | Missing the SalesLT schema |

---

## Slide 89: Homework / practice

![Slide 89](slides/slide-89.jpg)

- Rebuild Labs 1–7 without the recording
- Copy SalesLT.SalesOrderHeader to ADLS as Parquet into folder SalesOrderHeader
- Add a fileName parameter to ADLS_Generic_DS; name files <table>.csv
- Pipeline with 3 parallel copies + an On failure Wait/Set variable activity
- Bonus: folder per date with @formatDateTime(utcNow(),'yyyy/MM/dd')
- Compare the size of one table as CSV, JSON and Parquet

---

## Slide 90: Next session preview: Session 05: Databricks

![Slide 90](slides/slide-90.jpg)

- Big data concept
- Types of data
- Spark vs PySpark vs Databricks
- How to create a Databricks account
- Read a CSV file in Databricks
- Create a table in Databricks
- Genie in Databricks
- Lots of Q&A

---

## Slide 91: Thank you

![Slide 91](slides/slide-91.jpg)

- Questions?  ·  Publish your work  ·  Rebuild the 7 labs on your own

---

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)
