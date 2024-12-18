# Get started with querying a Kusto database in Microsoft Fabric

In Microsoft Fabric, an eventhouse is used to store real-time data related to events; often captured from a streaming data source by an eventstream.

Within an eventhouse, the data is stored in one or more KQL databases, each of which contains tables and other objects that you can query by using Kusto Query Language (KQL) or a subset of Structured Query Language (SQL).

In this exercise, you'll create and populate an eventhouse with some sample data related to taxi rides, and then query the data using KQL and SQL.

This exercise takes approximately 25 minutes to complete.

## Create a workspace
Before working with data in Fabric, create a workspace with the Fabric capacity enabled.

1. On the Microsoft Fabric home page at [https://app.fabric.microsoft.com/home?experience=fabric](https://app.fabric.microsoft.com/home?experience=fabric), select **Real-Time Intelligence**.
2. In the menu bar on the left, select **Workspaces** (the icon looks similar to 📇).
3. Create a new workspace with a name of **workspace46835074**, selecting a licensing mode in the **Advanced** section of type **Trial**.
4. When your new workspace opens, it should be empty.



## Create an Eventhouse
Now that you have a workspace with support for a Fabric capacity, you can create an eventhouse in it.

1. On the **Real-Time Intelligence** home page, create a new **Eventhouse** with a name of your choice.
2. When the eventhouse has been created, close any prompts or tips that are displayed until you see the eventhouse page:

![image](https://github.com/user-attachments/assets/5e3870ea-6c4e-4ca2-8985-bd9ed7c16629)




3. In the pane on the left, note that your eventhouse contains a KQL database with the same name as the eventhouse.
4. Select the **KQL database** to view it.

Currently there are no tables in the database. In the rest of this exercise you'll use an eventstream to load data from a real-time source into a table.

5. In the page for the KQL database, select **Get data > Sample**. Then choose the **Automotive operations analytics** sample data.
6. After the data is finished loading (which may take some time), verify that an **Automotive** table has been created.

![image](https://github.com/user-attachments/assets/1892146c-9ce0-4f49-8915-645138c6760a)

![image](https://github.com/user-attachments/assets/f4bbb521-1f25-44ff-87bd-d0ea4ba776c3)

![image](https://github.com/user-attachments/assets/70f97402-f3ea-4309-a6e3-e8de5cb47084)

![image](https://github.com/user-attachments/assets/5ec87b67-6ad2-4b7b-a1bf-d3a73e524036)





## Query data by using KQL
Kusto Query Language (KQL) is an intuitive, comprehensive language that you can use to query a KQL database.

### Retrieve data from a table with KQL
1. In the left pane of the eventhouse window, under your KQL database, select the default **queryset** file. This file contains some sample KQL queries to get you started.
2. Modify the first example query as follows:

```kql
Automotive
| take 100
```
> **NOTE**: The Pipe ( | ) character is used in KQL to separate query operators in a tabular expression.

3. Select the query code and run it to return 100 rows from the table.

![image](https://github.com/user-attachments/assets/84f62e40-0192-4be2-b27a-cd8c81f746de)


4. Run a query to retrieve specific columns using the `project` keyword:

```kql
// Use 'project' and 'take' to view a sample number of records in the table and check the data.
Automotive 
| project vendor_id, trip_distance
| take 10
```

NOTE: The use of // denotes a comment.


![image](https://github.com/user-attachments/assets/4faba485-2fe9-46db-8541-a5e31578ee96)

Another common practice in the analysis is renaming columns in our queryset to make them more user-friendly.
5. Rename a column for clarity:

```kql
Automotive 
| project vendor_id, ["Trip Distance"] = trip_distance
| take 10
```

![image](https://github.com/user-attachments/assets/8dac147d-c732-4ead-b09f-0e22ad50a8ae)


### Summarize data by using KQL
1. Use the `summarize` keyword to aggregate data:

```kql
Automotive
| summarize ["Total Trip Distance"] = sum(trip_distance)
```

![image](https://github.com/user-attachments/assets/b8b81f05-6306-43c9-b572-586133bee879)


2. Group the data by a specific column:

```kql
Automotive
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = pickup_boroname, ["Total Trip Distance"]
```

![image](https://github.com/user-attachments/assets/5ac9edaf-25c6-4f4f-9d5f-d356085e0740)


3. Handle blank or null values using the `case` function:

```kql
Automotive
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
```

![image](https://github.com/user-attachments/assets/316e87b1-0be0-49c6-bf3d-47983da0c427)


### Sort data by using KQL
1. Sort the summarized data alphabetically:

```kql
Automotive
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc
```

![image](https://github.com/user-attachments/assets/1ee9e37a-9197-406e-9bdf-572cc965385c)


2. Use `order by` instead of `sort by`:

```kql
Automotive
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| order by Borough asc
```

![image](https://github.com/user-attachments/assets/c6342a1a-1191-4136-8b34-b6773dcfbcdf)


### Filter data by using KQL
Filter records for a specific condition:

```kql
Automotive
| where pickup_boroname == "Manhattan"
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc
```

![image](https://github.com/user-attachments/assets/5ce44d15-0177-4f5a-b269-cc3b936ac0f1)


## Query data by using Transact-SQL
KQL Database supports a T-SQL endpoint, allowing you to query data using T-SQL syntax.

KQL Database doesn't support Transact-SQL natively, but it provides a T-SQL endpoint that emulates Microsoft SQL Server and allows you to run T-SQL queries on your data. The T-SQL endpoint has some limitations and differences from the native SQL Server. For example, it doesn't support creating, altering, or dropping tables, or inserting, updating, or deleting data. It also doesn't support some T-SQL functions and syntax that aren't compatible with KQL. It was created to allow systems that didn't support KQL to use T-SQL to query the data within a KQL Database. So, it's recommended to use KQL as the primary query language for KQL Database, as it offers more capabilities and performance than T-SQL. You can also use some SQL functions that are supported by KQL, such as count, sum, avg, min, max, and so on.

### Retrieve data using Transact-SQL
Run the following queries:

1. Retrieve the top 100 rows:

```sql
SELECT TOP 100 * FROM Automotive
```

![image](https://github.com/user-attachments/assets/b8749b94-e365-40ed-a326-0c74c523f93f)


2. Retrieve specific columns:

```sql
SELECT TOP 10 vendor_id, trip_distance FROM Automotive
```

![image](https://github.com/user-attachments/assets/73714d34-bb04-4b99-81cf-502fb6b65c64)


3. Rename a column:

```sql
SELECT TOP 10 vendor_id, trip_distance AS [Trip Distance] FROM Automotive
```

![image](https://github.com/user-attachments/assets/424ff895-fcfb-49d7-88cf-2db5a5b4b5f9)


### Summarize and group data
1. Find the total trip distance:

```sql
SELECT SUM(trip_distance) AS [Total Trip Distance] FROM Automotive
```

![image](https://github.com/user-attachments/assets/db288198-6a0b-435d-9601-562470c552fc)


2. Group by borough:

```sql
SELECT pickup_boroname AS Borough, SUM(trip_distance) AS [Total Trip Distance]
FROM Automotive
GROUP BY pickup_boroname
```

![image](https://github.com/user-attachments/assets/bd468b15-a587-4fc1-80da-a7e24af31f01)


3. Handle null values with a `CASE` statement:

```sql
SELECT CASE
         WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
         ELSE pickup_boroname
       END AS Borough,
       SUM(trip_distance) AS [Total Trip Distance]
FROM Automotive
GROUP BY CASE
           WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
           ELSE pickup_boroname
         END;
```

![image](https://github.com/user-attachments/assets/105bf55a-4730-48e3-8f97-31e3df4df002)


### Sort and filter data
Sort the results alphabetically and filter for Manhattan:

```sql
SELECT CASE
         WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
         ELSE pickup_boroname
       END AS Borough,
       SUM(trip_distance) AS [Total Trip Distance]
FROM Automotive
GROUP BY CASE
           WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
           ELSE pickup_boroname
         END
HAVING Borough = 'Manhattan'
ORDER BY Borough ASC;
```

![image](https://github.com/user-attachments/assets/069d177b-628e-4077-b381-e8874c243be4)


## Clean up resources
When you've finished exploring your KQL database, delete the workspace you created for this exercise.


## Congratulations
You have successfully completed this lab. Click **End** to mark the lab as Complete.
