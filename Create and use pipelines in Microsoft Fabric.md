# 📌 Ingest Data with a Pipeline in Microsoft Fabric

A **data lakehouse** is a common analytical data store for cloud-scale analytics solutions. One of the core tasks of a data engineer is to implement and manage **data ingestion** from multiple operational data sources into the lakehouse.  

In **Microsoft Fabric**, you can implement **ETL (Extract, Transform, Load)** or **ELT (Extract, Load, Transform)** solutions through pipelines. Fabric also supports **Apache Spark**, enabling large-scale data transformations. By combining pipelines and Spark, you can ingest data from external sources into **OneLake** storage and perform custom transformations before loading it into tables for analysis.

> This lab takes approximately 45 minutes and requires access to a Microsoft Fabric tenant.

---

## 🏗️ Step 1: Create a Workspace

1. Navigate to the [Microsoft Fabric home page](https://app.fabric.microsoft.com/home?experience=fabric-developer) and sign in.  
2. In the menu bar on the left, select **Workspaces** (🗇 icon).  
3. Create a new workspace:
   - Choose a name of your choice.  
   - Select a licensing mode with Fabric capacity (Trial, Premium, or Fabric).  
4. The new workspace should be empty upon opening.  

---

## 🏞️ Step 2: Create a Lakehouse

1. In the left menu, select **Create → Lakehouse** under **Data Engineering**.  
2. Provide a unique name. Ensure **Lakehouse schemas (Public Preview)** is disabled.  
3. After creation, in the Explorer pane, create a subfolder under **Files** named `new_data`.

---

## 🚀 Step 3: Create a Pipeline

1. On the lakehouse **Home** page, select **Get Data → New Data Pipeline**. Name it `Ingest Sales Data`.

   <img width="1237" height="472" alt="image" src="https://github.com/user-attachments/assets/d33c3632-85b4-4c69-be54-5fa75775c834" />

3. Use the **Copy Data wizard**:
   - **Source**: HTTP  
     - URL: `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`  
     - Authentication: Anonymous  
   - **Destination**: Files → `new_data` → `sales.csv`  
   - **File format**: DelimitedText, comma-separated, first row as header
     

   <img width="1102" height="566" alt="image" src="https://github.com/user-attachments/assets/c6c52c69-e3fb-47cd-9fe4-bff2518c8c09" />

   <img width="1113" height="599" alt="image" src="https://github.com/user-attachments/assets/37654c02-37a0-4bd5-a6ae-1c13ee25c0ee" />

   <img width="1099" height="582" alt="image" src="https://github.com/user-attachments/assets/3c23eb8d-7005-4979-a754-3ad5f02856ba" />

   <img width="1110" height="592" alt="image" src="https://github.com/user-attachments/assets/c795abba-7017-4a24-b64c-48f45917c4fc" />

   <img width="1130" height="623" alt="image" src="https://github.com/user-attachments/assets/978f03be-73e9-45b7-aced-dfc32a00ebbb" />

   <img width="1100" height="582" alt="image" src="https://github.com/user-attachments/assets/204a7c0f-838d-4e76-9421-96a2f685002b" />

   <img width="1109" height="582" alt="image" src="https://github.com/user-attachments/assets/0ea1928c-a9e9-4c35-9ca0-087c9c963a2b" />

   <img width="1101" height="583" alt="image" src="https://github.com/user-attachments/assets/647e1cd9-efb6-4403-8fd7-3689782bb5c3" />

     
4. **Save & Run** the pipeline. Monitor status until it succeeds.

   <img width="1281" height="647" alt="image" src="https://github.com/user-attachments/assets/52fbfa9b-6d25-4108-8292-4cbb99123cda" />



---

## 📓 Step 4: Create a Notebook

1. On the lakehouse **Home** page, select **Open Notebook → New Notebook**.
<img width="1131" height="481" alt="image" src="https://github.com/user-attachments/assets/e6fadc50-d3c7-47e6-ae20-154859d39fa9" />

   
3. Replace the default cell code with:

```python
table_name = "sales"
````

> Mark the cell as a **parameters cell**.
<img width="1297" height="546" alt="image" src="https://github.com/user-attachments/assets/2a437fe9-3dc9-4266-a094-9397c29d4638" />


3. Add a new code cell with the following Spark code:

```python
from pyspark.sql.functions import *

# Read the new sales data
df = spark.read.format("csv").option("header","true").load("Files/new_data/*.csv")

# Add month and year columns
df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

# Derive FirstName and LastName columns
df = df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0))\
       .withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

# Filter and reorder columns
df = df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", 
        "FirstName", "LastName", "EmailAddress", "Item", "Quantity", "UnitPrice", "TaxAmount"]

# Load the data into a table
df.write.format("delta").mode("append").saveAsTable(table_name)
```

4. Run all cells. Refresh Explorer → **Tables** to verify the `sales` table.

   <img width="1265" height="548" alt="image" src="https://github.com/user-attachments/assets/7826fe38-f9f3-4031-a6f2-f5080597d29e" />

   <img width="1312" height="619" alt="image" src="https://github.com/user-attachments/assets/2ee9b2c1-a731-4711-8e53-15e057420892" />




---

## 🔄 Step 5: Modify the Pipeline

1. Open the `Ingest Sales Data` pipeline.
2. Add a **Delete Data** activity **before** Copy Data:

   * Path: `Files/new_data/*.csv`
   * Recursively: Selected

     <img width="969" height="626" alt="image" src="https://github.com/user-attachments/assets/5b44a873-44d8-4a1d-85d4-a3c41001cdec" />


3. Add a **Notebook** activity after Copy Data:

   * Notebook: `Load Sales`
   * Base parameter: `table_name = new_sales`
4. Save & Run the pipeline. Check the `new_sales` table in the lakehouse.

   <img width="1217" height="666" alt="image" src="https://github.com/user-attachments/assets/c07b6ba8-1869-4d8a-b524-7484017cdde8" />



> This creates a reusable ETL process combining **data ingestion** and **Spark transformations**.

---

## ✅ Conclusion

You have successfully implemented a **data ingestion pipeline** in Microsoft Fabric:

* Copied data from external sources to a lakehouse.
* Used a **Spark notebook** to transform and load the data.
* Created a reusable **ETL pipeline** combining Copy Data, Delete Data, and Notebook activities.

This demonstrates **cloud-scale data engineering skills** in Microsoft Fabric, including pipelines, Spark transformations, and lakehouse management.

```
