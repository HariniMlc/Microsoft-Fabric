# 🏞️ Create a Microsoft Fabric Lakehouse

This project demonstrates how to create and work with a **Microsoft Fabric Lakehouse**, combining the flexibility of a data lake with the structure and performance of a data warehouse.

The implementation follows a hands-on lab from Microsoft Learn and focuses on workspace creation, data ingestion, table creation, and querying using SQL and visual queries.

📘 **Reference:**  
  - https://learn.microsoft.com/en-us/training/modules/get-started-lakehouses/
  - https://microsoftlearning.github.io/mslearn-fabric/Instructions/Labs/01-lakehouse.html

---

## 📌 What is a Lakehouse?

Traditional analytics solutions relied heavily on **data warehouses**, where data is stored in structured relational tables and queried using SQL.  
With the rise of **big data**, data lakes became popular due to their ability to store large volumes of raw data without enforcing schemas.

A **Lakehouse** combines both approaches:
- Data is stored as files in a data lake
- A relational schema is applied using metadata
- Data can be queried using SQL

In **Microsoft Fabric**, lakehouses use:
- **OneLake** (built on Azure Data Lake Storage Gen2)
- **Delta Lake** format for tables
- Built-in **SQL analytics endpoints**

---

## 🎯 Objectives

By completing this project, I learned how to:

- Create a workspace in Microsoft Fabric
- Create a Lakehouse
- Upload files to OneLake
- Load file data into Delta tables
- Query data using SQL
- Create visual queries using Power Query
- Understand Lakehouse file and table structures

---

## 🛠 Prerequisites

- Microsoft Fabric Trial / Premium / Fabric capacity
- Basic understanding of SQL and data concepts
- Web browser access to Microsoft Fabric

⏱️ **Estimated time:** ~30 minutes

---

## 🧱 Step-by-Step Implementation

### 1️⃣ Create a Workspace

1. Navigate to:  
   https://app.fabric.microsoft.com/home?experience=fabric
2. Sign in with Fabric credentials
3. Select **Workspaces** from the left menu
4. Create a new workspace
5. Choose a licensing mode that includes **Fabric capacity**
   <img width="727" height="493" alt="image" src="https://github.com/user-attachments/assets/525e1c82-cace-4cd8-99af-19324a93c4a1" />

7. Open the workspace (initially empty)

   <img width="613" height="1023" alt="image" src="https://github.com/user-attachments/assets/140ccb1c-ee2e-44b4-be51-f3d679a0ce8e" />


---

### 2️⃣ Create a Lakehouse

1. Select **Create** from the left menu
2. Under **Data Engineering**, choose **Lakehouse**
   <img width="737" height="563" alt="image" src="https://github.com/user-attachments/assets/13e43d21-821a-485d-a0f2-8fe7cfbdd66d" />

4. Provide a unique lakehouse name
   <img width="1014" height="697" alt="image" src="https://github.com/user-attachments/assets/6b99b34b-e574-463d-aba4-259194c55a20" />

6. Ensure **Lakehouse schemas (Public Preview)** is disabled
7. Create the Lakehouse

📂 The Lakehouse explorer contains:
- **Tables** → Managed Delta tables (SQL-queryable)
- **Files** → Raw data files stored in OneLake

---

### 3️⃣ Upload Data Files

1. Download the sample dataset:  
   https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv
2. In the Lakehouse **Files** folder:
   - Create a new subfolder called `data`
  
     <img width="520" height="505" alt="image" src="https://github.com/user-attachments/assets/6127557a-62d4-4cb9-beaf-79fe17858aa7" />

3. Upload `sales.csv` into the `data` folder
   <img width="687" height="462" alt="image" src="https://github.com/user-attachments/assets/895b2a05-d256-4491-80fc-b94579776570" />

5. Preview the file to verify successful upload

   <img width="1025" height="432" alt="image" src="https://github.com/user-attachments/assets/501df9d9-eedd-49c4-882f-af7f767dc97b" />


---

### 4️⃣ Explore Shortcuts (Optional)

Microsoft Fabric allows creating **shortcuts** to external data sources without copying data.

- From the **Files** folder, select **New shortcut**
- Review available data source options
- Close without creating a shortcut

  <img width="1035" height="782" alt="image" src="https://github.com/user-attachments/assets/f8d61d28-c6ed-4647-9591-ed293da36258" />


(This step helps understand external data integration options.)

---

### 5️⃣ Load File Data into a Table

1. Navigate to `Files/data`
2. Open the **…** menu for `sales.csv`
3. Select **Load to Tables → New table**
   <img width="1039" height="445" alt="image" src="https://github.com/user-attachments/assets/7b531eb0-3597-49ba-a3e7-ea8c99a72708" />

5. Name the table: `sales`
6. Confirm and wait for table creation
   <img width="720" height="496" alt="image" src="https://github.com/user-attachments/assets/b7c4037d-87c0-45dc-8abd-1c617136224a" />

8. Refresh the **Tables** folder if needed
9. Preview the table data
    <img width="830" height="724" alt="image" src="https://github.com/user-attachments/assets/d437e837-3682-4c7f-bd10-f4eceabccc74" />

    <img width="1016" height="341" alt="image" src="https://github.com/user-attachments/assets/2ed87f91-f511-4523-831b-9f4ad46de5f3" />


📌 The underlying table files are stored in:
- **Parquet format**
- A `_delta_log` folder for transaction history

  <img width="1016" height="341" alt="image" src="https://github.com/user-attachments/assets/d4784af1-d638-49ee-bd6b-df8b1d9fa508" />

---

### 6️⃣ Query Data Using SQL

1. Switch from **Lakehouse** view to **SQL analytics endpoint**
   
   <img width="373" height="250" alt="image" src="https://github.com/user-attachments/assets/a05bbb2f-a909-42ca-999d-4e32096a863d" />

3. Open a **New SQL query**
   
   <img width="451" height="602" alt="image" src="https://github.com/user-attachments/assets/d9758bdc-438c-44f4-9607-a1af6bfac486" />

5. Run the following query:

```sql
SELECT Item, 
       SUM(Quantity * UnitPrice) AS Revenue
FROM sales
GROUP BY Item
ORDER BY Revenue DESC;
```

<img width="1132" height="717" alt="image" src="https://github.com/user-attachments/assets/11dfe8b2-f59d-4190-af30-10fd35d46604" />



### Create a visual query
While many data professionals are familiar with SQL, data analysts with Power BI experience can apply their Power Query skills to create visual queries

1. On the toolbar, expand the New SQL query option and select New visual query.

<img width="397" height="247" alt="image" src="https://github.com/user-attachments/assets/1cf2fd74-f879-4ba9-a9cc-83fb5ff8a87e" />

2. Drag the sales table to the new visual query editor pane that opens to create a Power Query as shown here:

<img width="1152" height="777" alt="image" src="https://github.com/user-attachments/assets/afd7eccf-d807-4a2b-8b6b-ad6d9718bac8" />

3. In the Manage columns menu, select Choose columns. Then select only the SalesOrderNumber and SalesOrderLineNumber columns.

<img width="648" height="558" alt="image" src="https://github.com/user-attachments/assets/4fcc0628-4fef-469f-9693-00c405da0b31" />

4. in the Transform menu, select Group by. Then group the data by using the following Basic settings:

 - Group by: SalesOrderNumber
 - New column name: LineItems
 - Operation: Count distinct values
 - Column: SalesOrderLineNumber
  When you’re done, the results pane under the visual query shows the number of line items for each sales order.

<img width="762" height="605" alt="image" src="https://github.com/user-attachments/assets/74ff5b9d-be7a-4e7c-a4a4-2fc208ff8afc" />



In this exercise, you have created a lakehouse and imported data into it. You’ve seen how a lakehouse consists of files and tables stored in a OneLake data store. The managed tables can be queried using SQL, and are included in a default semantic model to support data visualizations.


