# 🔥 Analyze Data with Apache Spark in Microsoft Fabric

This project demonstrates how to analyze, transform, and visualize data using **Apache Spark** in **Microsoft Fabric** with a **Lakehouse** architecture.

The implementation is based on a hands-on Microsoft Fabric lab and showcases working with CSV files, Spark DataFrames, Delta tables, SQL, and data visualization.

📘 **References**
- Microsoft Learn:  
  https://learn.microsoft.com/en-us/training/modules/use-apache-spark-work-files-lakehouse/
- Lab Instructions:  
  https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/02-analyze-spark.html

---

## 🧠 Overview

Apache Spark in Microsoft Fabric enables scalable analytics on data stored in OneLake.  
In this project, Spark notebooks are used to:

- Load raw CSV files into Spark DataFrames
- Apply schemas and transformations
- Aggregate and analyze data
- Store results in Parquet and Delta formats
- Query data using Spark SQL
- Visualize insights using built-in charts, matplotlib, and seaborn

---

## 🎯 Learning Objectives

- Create and use Spark notebooks in Fabric
- Load multiple CSV files into a single DataFrame
- Apply schemas and data types
- Filter, group, and aggregate data
- Transform and partition data files
- Create Delta tables and query them using SQL
- Visualize analytical results

---

## 🛠 Prerequisites

- Microsoft Fabric workspace (Trial / Premium / Fabric capacity)
- Basic knowledge of Python and SQL
- Familiarity with data analytics concepts

---

## 🧱 Step-by-Step Implementation

### 1️⃣ Create Lakehouse and Upload Files

1. Create a **Lakehouse** in the Fabric workspace
2. Download and extract data files:  
   https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip
3. Upload the `orders` folder containing:
   - `2019.csv`
   - `2020.csv`
   - `2021.csv`
4. Verify files under **Files → orders**

---

### 2️⃣ Create a Spark Notebook

1. Create a new **Notebook** under Data Engineering
2. Rename it to something descriptive (e.g., `Sales_Order_Analysis`)
3. Convert the first cell to Markdown and add:

```md
# Sales order data exploration
Use this notebook to explore sales order data
````

---

### 3️⃣ Create a DataFrame from CSV Files

Load a single CSV file:

```python
df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
```

Define a schema for meaningful column names:

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

df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")
display(df)
```

✔️ This loads **2019–2021** data into a single DataFrame.

---

### 4️⃣ Explore and Filter Data

```python
customers = df.select("CustomerName", "Email")
print(customers.count())
print(customers.distinct().count())
display(customers.distinct())
```

Filter by product:

```python
customers = df.select("CustomerName", "Email") \
              .where(df['Item']=='Road-250 Red, 52')
display(customers.distinct())
```

---

### 5️⃣ Aggregate and Group Data

```python
productSales = df.select("Item", "Quantity") \
                 .groupBy("Item").sum()
display(productSales)
```
<img width="792" height="510" alt="image" src="https://github.com/user-attachments/assets/adbc0124-1e1f-49f8-a19c-6dcf8d3d54de" />



Sales by year:

```python
from pyspark.sql.functions import *

yearlySales = df.select(year(col("OrderDate")).alias("Year")) \
                .groupBy("Year").count().orderBy("Year")
display(yearlySales)
```

<img width="1052" height="490" alt="image" src="https://github.com/user-attachments/assets/1787a084-f301-4b30-98be-bdddffc52498" />


---

### 6️⃣ Transform Data

```python
transformed_df = df \
    .withColumn("Year", year(col("OrderDate"))) \
    .withColumn("Month", month(col("OrderDate"))) \
    .withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)) \
    .withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

display(transformed_df.limit(5))
```
<img width="995" height="716" alt="image" src="https://github.com/user-attachments/assets/b4a62836-8bde-4600-874c-1209e13831fa" />


---

### 7️⃣ Save Data as Parquet

```python
transformed_df.write.mode("overwrite") \
    .parquet("Files/transformed_data/orders")
```

Load it back:

```python
orders_df = spark.read.format("parquet") \
    .load("Files/transformed_data/orders")
display(orders_df)
```

---

### 8️⃣ Save Partitioned Data

```python
orders_df.write.partitionBy("Year","Month") \
    .mode("overwrite") \
    .parquet("Files/partitioned_data")
```

Load only 2021 data:

```python
orders_2021_df = spark.read.format("parquet") \
    .load("Files/partitioned_data/Year=2021/Month=*")
display(orders_2021_df)
```

---

### 9️⃣ Create Delta Table and Use SQL

```python
df.write.format("delta").saveAsTable("salesorders")
```

Query using Spark SQL:

```sql
%%sql
SELECT YEAR(OrderDate) AS OrderYear,
       SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
FROM salesorders
GROUP BY YEAR(OrderDate)
ORDER BY OrderYear;
```

---

### 🔟 Visualize Data

**Built-in chart**

* Bar chart: Item vs Quantity (SUM)

**Matplotlib**

```python
from matplotlib import pyplot as plt
df_sales = df_spark.toPandas()
plt.bar(df_sales['OrderYear'], df_sales['GrossRevenue'])
plt.show()
```

**Seaborn**

```python
import seaborn as sns
sns.set_theme(style="whitegrid")
sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
plt.show()
```

---

## ✅ Key Takeaways

* Spark DataFrames enable powerful transformations on lakehouse data
* Parquet + partitioning improves performance and scalability
* Delta tables bring transactional reliability to data lakes
* Fabric supports SQL, Python, and visualization in one platform

---

## ⭐ Author

**Harini**
Lead Data Engineer | Microsoft Fabric | Apache Spark | Azure Analytics

```

---

### If you want next:
- 🔹 Separate **Notebook.md** + **README.md**
- 🔹 Add **screenshots placeholders**
- 🔹 Convert this into a **series of Fabric portfolio projects**
- 🔹 Align wording with **Azure / Fabric job descriptions**

Just tell me — we’ll build this into a **strong Fabric portfolio** 💪
```
