# Microsoft Fabric Lakehouse Medallion Architecture Exercise

This exercise guides you through creating a **medallion architecture** in a Microsoft Fabric Lakehouse using notebooks.

Estimated completion time: ~45 minutes

**Prerequisite:** Access to a Microsoft Fabric tenant.

---

## 1. Create a Workspace

1. Navigate to [Microsoft Fabric home](https://app.fabric.microsoft.com/home?experience=fabric-developer) and sign in.
2. In the left menu, select **Workspaces**.
3. Create a new workspace with a name of your choice.
4. Select a licensing mode that includes Fabric capacity (Trial, Premium, or Fabric).
5. Open your workspace; it should be empty.

---

## 2. Create a Lakehouse and Upload Data to Bronze Layer

1. In your workspace, select **+ New item** > **Lakehouse**, name it `Sales`, and disable "Lakehouse schemas".
2. Download the data file [orders.zip](https://github.com/MicrosoftLearning/dp-data/blob/main/orders.zip), extract it (2019.csv, 2020.csv, 2021.csv).
3. In the **Explorer** pane, create a folder named `bronze` inside `Files`.
4. Upload the three CSV files into the `bronze` folder.
   <img width="1215" height="559" alt="image" src="https://github.com/user-attachments/assets/e63973ed-61b8-4bb4-a209-a94838198013" />


---

## 3. Transform Data and Load to Silver Delta Table

### 3.1 Create a Notebook

1. Open **New notebook** from the bronze folder.
2. Rename it to `Transform data for Silver`.
3. Remove any commented-out starter code.

### 3.2 Load Bronze Data into DataFrame

```python
from pyspark.sql.types import *

orderSchema = StructType([
   StructField("SalesOrderNumber", StringType()),
   StructField("SalesOrderLineNumber", IntegerType()),
   StructField("OrderDate", DateType()),
   StructField("CustomerName", StringType()),
   StructField("Email", StringType()),
   StructField("Item", StringType()),
   StructField("Quantity", IntegerType()),
   StructField("UnitPrice", FloatType()),
   StructField("Tax", FloatType())
])

# Load CSV files
 df = spark.read.format("csv").option("header", "false").schema(orderSchema).load("Files/bronze/*.csv")

# Preview data
 display(df.head(10))
```

### 3.3 Add Data Validation and Cleanup Columns

```python
from pyspark.sql.functions import when, lit, col, current_timestamp, input_file_name

df = df.withColumn("FileName", input_file_name()) \
   .withColumn("IsFlagged", when(col("OrderDate") < '2019-08-01', True).otherwise(False)) \
   .withColumn("CreatedTS", current_timestamp()) \
   .withColumn("ModifiedTS", current_timestamp())

# Update null or empty CustomerName
 df = df.withColumn("CustomerName", when((col("CustomerName").isNull() | (col("CustomerName")=="")), lit("Unknown")).otherwise(col("CustomerName")))
```

### 3.4 Create Silver Delta Table

```python
from delta.tables import *

DeltaTable.createIfNotExists(spark) \
   .tableName("sales_silver") \
   .addColumn("SalesOrderNumber", StringType()) \
   .addColumn("SalesOrderLineNumber", IntegerType()) \
   .addColumn("OrderDate", DateType()) \
   .addColumn("CustomerName", StringType()) \
   .addColumn("Email", StringType()) \
   .addColumn("Item", StringType()) \
   .addColumn("Quantity", IntegerType()) \
   .addColumn("UnitPrice", FloatType()) \
   .addColumn("Tax", FloatType()) \
   .addColumn("FileName", StringType()) \
   .addColumn("IsFlagged", BooleanType()) \
   .addColumn("CreatedTS", DateType()) \
   .addColumn("ModifiedTS", DateType()) \
   .execute()
```

### 3.5 Upsert Data into Silver Table

```python
deltaTable = DeltaTable.forPath(spark, 'Tables/dbo/sales_silver')
dfUpdates = df

deltaTable.alias('silver') \
 .merge(
   dfUpdates.alias('updates'),
   'silver.SalesOrderNumber = updates.SalesOrderNumber AND silver.OrderDate = updates.OrderDate AND silver.CustomerName = updates.CustomerName AND silver.Item = updates.Item'
 ) \
 .whenNotMatchedInsert(values = {
     "SalesOrderNumber": "updates.SalesOrderNumber",
     "SalesOrderLineNumber": "updates.SalesOrderLineNumber",
     "OrderDate": "updates.OrderDate",
     "CustomerName": "updates.CustomerName",
     "Email": "updates.Email",
     "Item": "updates.Item",
     "Quantity": "updates.Quantity",
     "UnitPrice": "updates.UnitPrice",
     "Tax": "updates.Tax",
     "FileName": "updates.FileName",
     "IsFlagged": "updates.IsFlagged",
     "CreatedTS": "updates.CreatedTS",
     "ModifiedTS": "updates.ModifiedTS"
 }) \
 .execute()
```

---

## 4. Explore Silver Layer Using SQL Endpoint

1. Open the **Sales SQL analytics endpoint**.
   <img width="946" height="406" alt="image" src="https://github.com/user-attachments/assets/186fcdce-66a2-4ca3-b08c-d49a0e37e8f1" />

3. Create and run queries to explore data, e.g.:

```sql
-- Total sales per year
SELECT YEAR(OrderDate) AS Year,
       CAST(SUM(Quantity * (UnitPrice + Tax)) AS DECIMAL(12,2)) AS TotalSales
FROM sales_silver
GROUP BY YEAR(OrderDate)
ORDER BY YEAR(OrderDate);
```
<img width="1143" height="630" alt="image" src="https://github.com/user-attachments/assets/f74eecb3-6252-4f53-bef9-321ae2a449a7" />

```sql
-- Top 10 customers by quantity
SELECT TOP 10 CustomerName, SUM(Quantity) AS TotalQuantity
FROM sales_silver
GROUP BY CustomerName
ORDER BY TotalQuantity DESC;
```

<img width="663" height="481" alt="image" src="https://github.com/user-attachments/assets/deb29eb8-4c39-405f-8de5-ffb4a3b22a29" />


---

## 5. Transform Data for Gold Layer

### 5.1 Create Notebook `Transform data for Gold`

```python
# Load silver data
 df = spark.read.table("Sales.sales_silver")
```

### 5.2 Create Date Dimension (dimdate_gold)

```python
from pyspark.sql.functions import col, dayofmonth, month, year, date_format

DeltaTable.createIfNotExists(spark) \
   .tableName("sales.dimdate_gold") \
   .addColumn("OrderDate", DateType()) \
   .addColumn("Day", IntegerType()) \
   .addColumn("Month", IntegerType()) \
   .addColumn("Year", IntegerType()) \
   .addColumn("mmmyyyy", StringType()) \
   .addColumn("yyyymm", StringType()) \
   .execute()

dfdimDate_gold = df.dropDuplicates(["OrderDate"]).select(
   col("OrderDate"),
   dayofmonth("OrderDate").alias("Day"),
   month("OrderDate").alias("Month"),
   year("OrderDate").alias("Year"),
   date_format(col("OrderDate"), "MMM-yyyy").alias("mmmyyyy"),
   date_format(col("OrderDate"), "yyyyMM").alias("yyyymm")
).orderBy("OrderDate")
```

### 5.3 Create Customer Dimension (dimcustomer_gold)

* Drop duplicates, split names, create CustomerID, and upsert.

### 5.4 Create Product Dimension (dimproduct_gold)

* Drop duplicates, split item info, create ItemID, and upsert.

### 5.5 Create Fact Table (factsales_gold)

* Combine silver data with dimension tables to create a fact table.
* Upsert new sales records using Delta Lake merge.

---

## 6. Optional: Create Semantic Model

* Requires Power BI or Fabric F64 SKU.
* Create new semantic model `Sales_Gold`.
* Include gold tables: `dimdate_gold`, `dimcustomer_gold`, `dimproduct_gold`, `factsales_gold`.
* Build relationships and measures for reporting.

---

*End of Exercise: You now have a fully curated Medallion Architecture (Bronze → Silver → Gold) in Microsoft Fabric Lakehouse.*
