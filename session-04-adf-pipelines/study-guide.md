# Session 4: ADF Pipelines (Study guide)

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)

**What you will learn:** You will learn the three types of data and why data lakes prefer **Parquet** over CSV and JSON. You will use a **source query** to choose columns, filter rows and join tables, put **several Copy activities in one pipeline**, and control the order with the four **dependency conditions**. You will write each table into **its own folder** in the lake, and replace many hard-coded datasets with **one parameterised dataset**. Seven labs take you there step by step.

**Before you start:** Everything from Session 3: the resource group, the ADLS Gen2 account with its `landing` container, the Data Factory, the Azure SQL Database with the AdventureWorksLT sample, and the `SQL_DB_LS` and `ADLS_LS` linked services.

**Slides:** [slides.md](slides.md) · [Download the slides as PDF](slides.pdf)

> **Names used in this guide**
>
> This guide reuses the names from Sessions 2 and 3. The screenshots use other example names (`rg-adf-training`, `adf-demo-dev`, `stdemoadls`, `sql-demo-server`, `sqldb-demo`): use your own.
>
> | Item | Name in this guide | Created in |
> | --- | --- | --- |
> | Resource group | `rg-session02-lab` | Session 2 |
> | ADLS Gen2 account and container | `stsession02adls` / `landing` | Session 2 |
> | Data Factory | `adf-session02-lab` | Session 2 |
> | SQL server and database | `sql-session03-lab` / `sqldb-session03` | Session 3 |
> | Linked services | `SQL_DB_LS`, `ADLS_LS` | Session 3 |
> | Datasets from Session 3 | `Customer_SQL_DB_DS`, `ADLS_CSV_DS`, `ADLS_JSON_DS` | Session 3 |
> | New datasets | `Customer_address_sql_DS`, `ADLS_Parquet_DS`, `Product_Folder_CSV_DS`, `ADLS_Generic_DS` | Today |
> | New pipelines | `Ingest_customer_address`, `Ingest_address_data`, `Joined_data`, `Multiple_table_ingestion`, `Customer_data_to_folder`, `Ingest_product_desc` | Today |

## Recap of Session 3

- **Linked service:** how to connect (server, database, credentials, storage URL).
- **Dataset:** what data, such as a table or a file or folder in a given format.
- **Copy activity:** reads the source dataset and writes the sink dataset.
- **Pipeline:** a group of activities, and the unit you run and monitor.
- **Integration runtime:** the compute. We use `AutoResolveIntegrationRuntime`.
- **Debug** runs the draft version. **Publish all** saves the work in Live mode.

![Recap of Session 3](slides/slide-03.jpg)

### Why this session matters

Last week you built one pipeline per table, with hard-coded datasets, and every file landed in one flat folder. That works for two tables. A real company may have 200 tables, and nobody wants 200 pipelines and 400 datasets. Today you learn the techniques that make pipelines scale: the right file format, source queries, several activities in one pipeline, dependency conditions, sink folders and dataset parameters.

![Today's labs at a glance](slides/slide-11.jpg)

## 1. Types of data and file formats

### Three types of data

![Three types of data](slides/slide-08.jpg)

| Type | What it is | Examples |
| --- | --- | --- |
| **Structured** | A fixed schema of rows and columns; easy to query with SQL | Relational tables, CSV files, Excel sheets |
| **Semi-structured** | Carries its structure with it (keys or tags); the schema can change from record to record and records can be nested | JSON, XML, log files |
| **Unstructured** | No schema at all; needs AI, OCR or speech-to-text to extract value | Images, video, audio, PDF documents, free text |

A data lake such as ADLS Gen2 stores all three types side by side. Our SQL tables are structured data; when ADF writes them to the lake, **you choose the file format**.

### CSV vs JSON vs Parquet

![CSV vs JSON vs Parquet](slides/slide-09.jpg)

| | CSV (DelimitedText) | JSON | Parquet |
| --- | --- | --- | --- |
| Storage | Text, one row per line | Text, one object per record | Binary, **columnar** |
| Column names | Once, in the header | Repeated in every record | Stored in the file's schema |
| Data types | None | Basic (string, number, boolean) | Full schema with data types |
| Compression | None built in | None built in | Very good (snappy by default) |
| Size | Small | Biggest | Smallest |
| Best for | Simple exchange, Excel | APIs, nested data | Analytics in a data lake |

**Delta Lake**, used by Databricks and Microsoft Fabric, is built on Parquet files plus a transaction log. In most companies the raw layer may be CSV or JSON, but the curated layers are almost always Parquet or Delta.

### Row storage vs columnar storage

![Row storage vs columnar storage](slides/slide-10.jpg)

- **Row-based** formats (CSV, JSON and OLTP databases) store a complete record together. That is ideal for reading or writing one record at a time.
- **Columnar** formats (Parquet, ORC) store all values of one column together. Analytical queries usually read a few columns over millions of rows, such as total sales per city, so the engine reads only the `City` and `Amount` columns and skips the rest. Values in one column are similar, so they compress very well.

This is the OLTP vs OLAP idea from Session 2, applied to files.

## 2. Lab 1: copy a table to ADLS Gen2 as Parquet

**Goal:** copy `SalesLT.CustomerAddress` to `landing/SalesLT.CustomerAddress.parquet`.

| # | Object | Type | Details |
| --- | --- | --- | --- |
| 1 | `Ingest_customer_address` | Pipeline | The job |
| 2 | `Customer_address_sql_DS` | Dataset | Azure SQL table `SalesLT.CustomerAddress` on `SQL_DB_LS` |
| 3 | `ADLS_Parquet_DS` | Dataset | Parquet in `landing` on `ADLS_LS` |
| 4 | CustomerAddress table to ADLS parquet | Copy activity | Source 2, sink 3 |

No new linked services are needed: you reuse `SQL_DB_LS` and `ADLS_LS`.

### Step 1: create the pipeline

1. In ADF Studio, open **Author** (pencil).
2. Next to **Factory Resources**, select **+** > **Pipeline** > **Pipeline**.
3. In the **Properties** pane, set **Name** to `Ingest_customer_address`.

![New pipeline](slides/slide-13.jpg)

### Step 2: add a Copy data activity

From **Activities** > **Move and transform**, drag **Copy data** onto the canvas. On the **General** tab, name it `CustomerAddress table to ADLS parquet`. The red numbers on the **Source** and **Sink** tabs mean required fields are still missing.

### Step 3: create the source dataset

1. On the **Source** tab, select **+ New**.
2. Search for `sql`, choose **Azure SQL Database** and select **Continue**.
3. **Name:** `Customer_address_sql_DS`. **Linked service:** `SQL_DB_LS`. **Import schema:** From connection/store.
4. In **Table name**, choose `SalesLT.CustomerAddress` (the link table between customers and addresses: `CustomerID`, `AddressID`, `AddressType`).
5. Select **OK**, then **Preview data** to check.

![Select SalesLT.CustomerAddress](slides/slide-17.jpg)

Creating a dataset from inside the activity selects it automatically. If the table list is slow, the serverless database may be resuming.

### Step 4: create the Parquet sink dataset

1. On the **Sink** tab, select **+ New**, search for `data lake` and choose **Azure Data Lake Storage Gen2**.
2. Choose the **Parquet** format and select **Continue**.
3. **Name:** `ADLS_Parquet_DS`. **Linked service:** `ADLS_LS`.
4. **File path:** browse to `landing`. Leave **Directory** and **File name** empty.
5. **Import schema:** **None**, because the file does not exist yet. Select **OK**.

![Choose the Parquet format](slides/slide-19.jpg)

![File path and import schema](slides/slide-21.jpg)

Other formats in the list: **Avro** is row-based binary (popular with streaming), **ORC** is another columnar format, **Iceberg** is an open table format, and **Binary** copies files as they are without reading them.

> **Tip:** *Import schema: From connection/store* reads an existing file. For a new sink file or folder, choose **None**.

### Step 5: debug and validate

1. Select **Debug** and watch the **Output** tab: Queued, In progress, Succeeded.
2. Hover over the activity row and select the glasses icon to open the copy details.

![Copy details](slides/slide-24.jpg)

- **Rows read = rows written = 417.** The Parquet file is about 21 KB.
- In the demo the database and the storage account were in different regions, so the data crossed regions. In a real project keep them in the **same region** to reduce latency and data-transfer cost.

3. In the storage account, open **Containers** > `landing`. The new file is named after the source table: `SalesLT.CustomerAddress.parquet`.

![The Parquet file in ADLS](slides/slide-25.jpg)

### Parquet is binary

If you open the file with **Edit**, you see unreadable characters and a warning that the file may not render correctly. **The file is not corrupted.** Parquet is binary and compressed, which is exactly why it is small and fast. Read it with Spark or Databricks, Synapse serverless SQL, Microsoft Fabric, Power BI or Python pandas. In Session 5 you read files like this with Databricks.

![Parquet is binary](slides/slide-26.jpg)

## 3. Source queries and joins

### Table, Query or Stored procedure?

The **Source** tab of the Copy activity has a **Use query** option:

| Option | What it copies |
| --- | --- |
| **Table** | The whole table: all rows, all columns |
| **Query** | Your own `SELECT`: choose columns, filter rows with `WHERE`, join tables, rename columns |
| **Stored procedure** | The rows returned by a procedure in the database |

![Table, Query or Stored procedure](slides/slide-28.jpg)

A query copies only the data you need: smaller files, lower cost, and no personal data you do not need. It is also the basis of **incremental loading** (`WHERE ModifiedDate > <last load date>`).

> **Important:** with **Query**, the table set in the dataset is **ignored**. ADF uses the dataset only for its linked service. That is why one SQL dataset can serve any query on the same database.

### Lab 2: query a table and write JSON

1. Create a pipeline named `Ingest_address_data` and add a **Copy data** activity named `Address table to ADLS`.
   - If ADF shows *Activity name should not have trailing space*, remove the space at the end of the name. It is invisible but still counts.
2. **Source dataset:** reuse any dataset on `SQL_DB_LS`, for example `Customer_SQL_DB_DS`.
3. Set **Use query** to **Query** and type:

   ```sql
   SELECT * FROM SalesLT.Address;
   ```

4. Select **Preview data**. You now see `AddressID`, `AddressLine1`, `City`, `StateProvince`, `CountryRegion`, `PostalCode`: the query result, not the dataset's table.
5. **Sink dataset:** `ADLS_JSON_DS` from Session 3. It has no file name, so it works for any source.
6. **Debug.** 450 rows are read and written, in about 24 seconds.

![Write the query](slides/slide-32.jpg)

![Debug and copy details](slides/slide-34.jpg)

**Validate:** the new file in `landing` is called `data_<guid>`, not `SalesLT.Address.json`. Each line is one JSON object, such as `{"AddressID":9,"AddressLine1":"8713 Yosemite Ct.",...}`. The JSON file is about twice the size of the data read, because every record repeats all the column names.

![The file is called data_guid](slides/slide-35.jpg)

### Where does my file go, and what is it called?

| Situation | Result |
| --- | --- |
| Table source, empty file name | File named after the table, for example `SalesLT.CustomerAddress.parquet` |
| Query source, empty file name | `data_<guid>.<ext>`: ADF has no table name to use |
| **Directory** set | ADF creates the folder(s) automatically during the copy |
| **File name** set | Exactly that name; running again overwrites it |
| Import schema on a folder that does not exist yet | Fails with *PathNotFound* |

![Where does my file go](slides/slide-36.jpg)

### Lab 3: join two tables

**Explore first.** Before you write a join, look at the data. Open the database > **Query editor (preview)** and sign in. If you see *Client with IP address ... is not allowed*, select **Allowlist IP** (or add your IP under the server's **Networking** page). Then run:

```sql
SELECT * FROM SalesLT.CustomerAddress;
```

![Explore the tables](slides/slide-37.jpg)

A **JOIN** combines rows from two tables that share a key. `Customer` and `CustomerAddress` share `CustomerID`.

![Joining two tables](slides/slide-38.jpg)

1. Create a pipeline named `Joined_data` with a **Copy data** activity and a SQL source dataset.
2. Set **Use query** to **Query** and try:

   ```sql
   select * from SalesLT.Customer c
   join SalesLT.CustomerAddress cd on c.CustomerID = cd.CustomerID;
   ```

3. **Preview data** fails with error **9019**: *There are duplicate column names CustomerID in the structure.*

![Duplicate column names](slides/slide-40.jpg)

**Why?** `SELECT *` on a join returns all columns of both tables. Both tables have `CustomerID`, `rowguid` and `ModifiedDate`. A database result can contain duplicate names, but a file cannot.

**Fix:** list the columns you need, with table aliases, and rename clashes with `AS`:

```sql
SELECT c.CustomerID, c.FirstName, cd.AddressType
FROM SalesLT.Customer c
JOIN SalesLT.CustomerAddress cd ON c.CustomerID = cd.CustomerID;
-- if you need both dates: c.ModifiedDate AS CustomerModifiedDate, cd.ModifiedDate AS AddressModifiedDate
```

4. **Preview data** now shows `CustomerID`, `FirstName` and `AddressType`. Choose a sink dataset (CSV or Parquet), **Debug**, and check the file in `landing`.

> **Tip:** Never use `SELECT *` on a join in a pipeline. List the columns.

## 4. Many tables in one pipeline

### The four dependency conditions

Activities are connected with **dependency arrows**. Each activity has four output handles on its right edge:

| Condition | Arrow | The next activity runs ... | Typical use |
| --- | --- | --- | --- |
| **On success** | Green | Only if this activity succeeded | The normal flow |
| **On failure** | Red | Only if this activity failed | Send an alert, write an error log, clean up |
| **On completion** | Blue | Whatever the result | Steps that must always run, such as a log row |
| **On skip** | Grey | Only if this activity was skipped | Rare; when an earlier condition was not met |

![The four dependency conditions](slides/slide-43.jpg)

Example: copy the sales table; **on success** refresh the report; **on failure** email the team. To change a condition, select the arrow (or right-click it).

### Lab 4: three tables in a chain

1. Create a pipeline named `Multiple_table_ingestion`.
2. Add a **Copy data** activity named `Customer Data`. **Source:** `Customer_SQL_DB_DS` (Table). **Sink:** `ADLS_CSV_DS`.
3. Add a second **Copy data** activity named `Address data`. Drag the **green** handle of `Customer Data` onto it.
   - **Source:** Use query = Query, `select * from SalesLT.Address`. **Sink:** `ADLS_CSV_DS`.
4. Add a third activity named `Product data`, connected **On success** from `Address data`.
   - **Source:** Use query = Query, `select * from SalesLT.Product`. **Sink:** `ADLS_CSV_DS`.
5. **Debug.** The activities start one after another, each taking about 25 seconds: **about 80 seconds in total**.

![The four handles in ADF Studio](slides/slide-47.jpg)

![Run the chain](slides/slide-52.jpg)

### Sequential or parallel?

If activities are **not connected**, ADF starts them all at the same time. The pipeline then takes about as long as the slowest activity.

![Sequential or parallel](slides/slide-53.jpg)

6. Select each arrow and delete it. **Debug** again: all three activities start at the same second and the pipeline finishes in **about 27 seconds instead of 80**.

![Parallel result and the files](slides/slide-55.jpg)

- **Chain** activities only when one depends on another, for example dimensions before facts, or copy before transform.
- Run **independent** tables in parallel. Remember that the source database must handle several queries at once.

**The problem:** look at `landing`. The files are called `data_03e1f260...`, `data_5d1cf613...`. With a query source and a shared sink dataset, every file lands in the same folder with a generated name, and nobody can tell which file belongs to which table. Lab 5 solves this.

## 5. Lab 5: one folder per table

1. Create a pipeline named `Customer_data_to_folder` with a **Copy data** activity. **Source query:** `select * from SalesLT.Customer`.
2. **Sink:** `ADLS_CSV_DS` > **Open**. In **File path**, type `Customer` in the **Directory** field. ADF creates `landing/Customer/` during the copy; you do not create it first.

![Set a directory on the sink dataset](slides/slide-58.jpg)

> **Warning:** `ADLS_CSV_DS` is also used by your Session 3 pipeline and by Lab 4. Hard-coding a folder in a **shared** dataset changes **every** pipeline that uses it. This is one reason for parameters (Lab 6). Clear the **Directory** again when you have finished this lab.

3. Add a second activity named `Product data`, connected **On success**. For its sink, select **+ New** > **Azure Data Lake Storage Gen2** > **DelimitedText**.
4. **Name:** `Product_Folder_CSV_DS`. **Linked service:** `ADLS_LS`. Keep **First row as header** ticked. **File path:** `landing` / `Product`.
5. If **Import schema** is *From connection/store*, you get *Schema import failed ... PathNotFound: The specified path does not exist*, because the folder does not exist yet. Choose **None** and select **OK**.

![Schema import error](slides/slide-61.jpg)

6. Set **Source query** for `Product data` to `select * from SalesLT.Product`, then **Debug**.
7. Refresh `landing`: there is now a `Customer` folder and a `Product` folder, each with one CSV file with a header row.

![One folder per table](slides/slide-63.jpg)

**It works, but** you needed one hard-coded dataset per folder. A hundred tables would need a hundred datasets. Lab 6 replaces them with one.

## 6. Lab 6: one parameterised dataset

### What is a parameter?

A **parameter** is a placeholder whose value is supplied when the pipeline runs. Instead of one dataset per folder, you create **one** dataset with a `folderName` parameter, and each Copy activity passes its own value.

![Hard-coded vs parameterised datasets](slides/slide-67.jpg)

Parameters exist at several levels: **linked service**, **dataset**, **pipeline**, and **global** parameters for the whole factory. Combined with the **Lookup** and **ForEach** activities (later in the course), they let one pipeline copy any number of tables from a list: a **metadata-driven** pipeline.

### How a dataset parameter works

![How a dataset parameter works](slides/slide-68.jpg)

1. In the dataset, **Parameters** tab > **+ New**: `folderName`, type **String**.
2. In the **Connection** tab, select the **Directory** box > **Add dynamic content** > choose `folderName`. The expression is `@dataset().folderName`.
3. In the Copy activity **Sink** tab, choose the dataset. A **Dataset properties** section asks for `folderName`. Type a value.
4. At run time ADF evaluates the expression and writes the file to `landing/<value>/`.

Expressions always start with `@`. Inside a dataset use `@dataset().<name>`; inside a pipeline use `@pipeline().parameters.<name>`.

### Step by step

1. Create a new **Azure Data Lake Storage Gen2** dataset with the **DelimitedText** format.
2. **Name:** `ADLS_Generic_DS`. **Linked service:** `ADLS_LS`. **File path:** `landing`, with **Directory** and **File name** empty. **Import schema:** None.
3. Open the **Parameters** tab, select **+ New**, and create `folderName` of type **String** with no default value.

   ![Create the folderName parameter](slides/slide-72.jpg)

4. Go back to **Connection**, select the **Directory** box, then **Add dynamic content** (Alt+Shift+D).
5. In the expression builder, select `folderName` under **Parameters**. The box shows `@dataset().folderName`. Select **OK**.

   ![Pipeline expression builder](slides/slide-73.jpg)

6. In `Customer_data_to_folder`, select `Customer data` and set its sink to `ADLS_Generic_DS`. Under **Dataset properties**, set `folderName` to `Customer_Folder`.
7. Select `Product data`, set its sink to the **same** `ADLS_Generic_DS`, and set `folderName` to `Product_Folder`.

   ![Pass a value from each activity](slides/slide-75.jpg)

8. **Debug.** Both activities succeed, in about 26 seconds each.
9. Refresh `landing`: `Customer_Folder` and `Product_Folder` were created from **one** dataset. A new table now needs only a new activity and a new value, not a new dataset.

![New folders from one dataset](slides/slide-77.jpg)

## 7. Lab 7: practice on your own

**Task:** copy `SalesLT.ProductDescription` into its own folder in `landing`, using `ADLS_Generic_DS`.

1. Create a pipeline named `Ingest_product_desc` with one **Copy data** activity.
2. **Source:** Use query = Query, `select * from productdescription`.
3. **Sink:** `ADLS_Generic_DS`, with `folderName` = `ProductDescription`.
4. **Debug.** The run **fails**.

![The run fails](slides/slide-81.jpg)

**Read the error:** *Error code 2200, User configuration issue. Failure happened on Source side ... Invalid object name 'productdescription'.*

**Why?** The table exists, but in the `SalesLT` schema. Without a schema prefix, SQL Server looks for `dbo.productdescription`, which does not exist.

**Fix:**

```sql
select * from SalesLT.ProductDescription;
```

5. **Debug** again. The activity succeeds in about 27 seconds, and a `ProductDescription` folder appears in `landing` with one file of about 145 KiB.
6. Select **Publish all** to save every pipeline and dataset you created today.

![Validate the new folder](slides/slide-83.jpg)

## 8. Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| *Activity name should not have trailing space* | A space at the end of the name | Remove the space |
| *Duplicate column names* (9019) | `SELECT *` on a join | List the columns; rename clashes with `AS` |
| *Schema import failed: PathNotFound* | The sink folder does not exist yet | Set **Import schema** to **None** |
| *Invalid object name* (2200) | Missing schema (`SalesLT.`) or a typo | Use `schema.table` |
| *Client with IP address ... is not allowed* (Query editor) | No firewall rule, or your IP changed | **Allowlist IP** on the server |
| Files named `data_<guid>` | Query source and no file name | Set a folder or a file name |
| Other pipelines suddenly write to a new folder | A shared dataset was edited | Use a parameterised dataset |
| The Parquet file looks corrupted in **Edit** | Parquet is binary | Read it with Spark, Synapse or Fabric |
| The first connection takes up to a minute | The serverless database is resuming | Wait and try again |

## 9. Best practices

- Use **Parquet** (or Delta) for analytics; keep CSV and JSON for exchange.
- **Copy only what you need:** explicit columns and `WHERE` filters.
- Never `SELECT *` on a join, and always **prefix tables with the schema**.
- Use **one folder per table**, and later one per load date. A typical layout is `landing/<source>/<table>/<yyyy>/<MM>/<dd>/file.parquet`, which you can build with expressions such as `@formatDateTime(utcNow(),'yyyy/MM/dd')`.
- **Parameterise datasets** instead of creating one dataset per table.
- Run **independent** copies in parallel; chain only real dependencies.
- Use **On failure** for alerts and logging.

## Wrap-up and next steps

### Key takeaways

- **Structured, semi-structured, unstructured:** a lake can hold all three.
- **Parquet** is binary, columnar, typed and compressed: the standard for analytics. It is not human-readable, and that is normal.
- A **source query** copies only what you need. With a query, the dataset's table is ignored and the file is named `data_<guid>` unless you set a name.
- **Dependency conditions:** success (green), failure (red), completion (blue), skip (grey). Unconnected activities run **in parallel**.
- Setting a **Directory** on the sink creates the folder at run time. Use **Import schema: None** for folders that do not exist yet.
- A **dataset parameter** (`@dataset().folderName`) turns many hard-coded datasets into one reusable dataset.

### Prepare before Session 5

Session 5 is **Databricks**:

- Big data concept
- Types of data
- Spark vs PySpark vs Databricks
- How to create a Databricks account
- Read a CSV file in Databricks
- Create a table in Databricks
- Genie in Databricks
- Lots of Q&A

Before Session 5:

- Make sure today's pipelines are **published** and that `landing` contains your Parquet file and your table folders. You will read files like these in Databricks.
- Keep your Azure resources. The serverless database auto-pauses and ADF costs nothing while idle.
- Bring your questions: Session 5 has a long Q&A block.

## Practice: quiz and assignment

### Quiz

1. Which format is binary and columnar?
   - A. CSV
   - B. JSON
   - C. Parquet
   - D. XML
2. Is a JSON file structured, semi-structured or unstructured? Why?
3. Why does columnar storage compress better than row storage?
4. Which **Use query** option lets you join two tables?
5. You chose **Use query: Query**. Which table in the dataset is used?
6. A Copy activity with a query source and an empty file name writes a file. What is it called?
7. Your join query fails with *duplicate column names CustomerID*. What is the cause, and how do you fix it?
8. Which dependency condition runs the next activity only if the previous one failed?
   - A. On success
   - B. On failure
   - C. On completion
   - D. On skip
9. Three Copy activities in one pipeline are not connected. How do they run?
10. Schema import on a new sink folder fails with *PathNotFound*. What should you do?
11. Write the expression that reads a dataset parameter called `folderName`.
12. Why did `select * from productdescription` fail with *Invalid object name*?

<details>
<summary>Show the answers</summary>

1. C. Parquet stores data by column in a binary, compressed file.
2. Semi-structured. Each record carries its own structure (keys), the fields can differ between records, and records can be nested.
3. All values of one column are stored together and are similar (same type, often repeated), so they compress much better than mixed values in a row.
4. **Query.** You write your own `SELECT` with a `JOIN`.
5. None. With a query, the dataset's table is ignored; ADF only uses the dataset's linked service.
6. `data_<guid>` with the format's extension. ADF has no table name to use.
7. `SELECT *` returns `CustomerID` from both tables, and a file cannot have two columns with the same name. List the columns explicitly and rename clashes with `AS`.
8. B. On failure.
9. In parallel: they all start at the same time.
10. Set **Import schema** to **None**. ADF creates the folder during the copy.
11. `@dataset().folderName`
12. The table is in the `SalesLT` schema. Without a prefix, SQL Server looks in `dbo`. Use `SalesLT.ProductDescription`.

</details>

### Assignment

1. Rebuild Labs 1 to 7 in your own factory, with your own names, without watching the recording.
2. Copy `SalesLT.SalesOrderHeader` to ADLS Gen2 as **Parquet**, into a folder named `SalesOrderHeader`.
3. Add a second parameter, `fileName`, to `ADLS_Generic_DS`, use it in the **File name** box, and name the files `<table>.csv`.
4. Build a pipeline with **three parallel** copies and an **On failure** activity (for example **Wait** or **Set variable**) after one of them.
5. Bonus: write into a folder per date with `@formatDateTime(utcNow(),'yyyy/MM/dd')`.
6. Compare the size of the same table as CSV, JSON and Parquet.

**What to submit:** A screenshot of your `landing` container showing the folders, and a screenshot of one successful pipeline run. Share them the way your instructor asks (for example in the course group), and hide your subscription ID and email address. Never include passwords, access keys, connection strings or card details.

## Key terms

| Term | Meaning |
| --- | --- |
| Structured data | Data with a fixed schema of rows and columns |
| Semi-structured data | Data that carries its own structure with keys or tags, such as JSON or XML |
| Unstructured data | Data with no schema, such as images, video, audio and PDF |
| Parquet | A binary, columnar, compressed file format with an embedded schema |
| Columnar storage | Storing all values of one column together |
| Delta Lake | Parquet files plus a transaction log (ACID, time travel); used by Databricks and Fabric |
| Use query | The Copy activity source option: Table, Query or Stored procedure |
| JOIN | A SQL operation that combines rows from two tables on a shared key |
| Dependency condition | The rule on an arrow between activities: success, failure, completion or skip |
| Sequential / parallel | Activities run one after another (connected) or at the same time (not connected) |
| Directory | The folder part of a dataset's file path in ADLS |
| Import schema | Reading column names and types from the source or sink when creating a dataset |
| Parameter | A placeholder whose value is supplied at run time |
| Dynamic content | An expression (starting with `@`) evaluated at run time |
| `@dataset().name` | The expression that reads a dataset parameter |
| Metadata-driven pipeline | One pipeline that copies many tables from a list, using parameters, Lookup and ForEach |
| Incremental load | Copying only rows changed since the last load, for example with `WHERE ModifiedDate > ...` |
| Schema (SQL) | A namespace for tables in a database, such as `SalesLT` or `dbo` |

<a href="https://www.linkedin.com/in/uwayo/"><img src="../assets/linkedin.png" alt="LinkedIn" height="16"></a> **Prepared by Ir. UWAYO Jacques** · [Follow me on LinkedIn](https://www.linkedin.com/in/uwayo/)
