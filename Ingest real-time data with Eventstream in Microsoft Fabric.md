# Microsoft Fabric Eventstream Lab

**Eventstream** is a feature in Microsoft Fabric that captures, transforms, and routes real-time events to various destinations. You can add event data sources, destinations, and transformations to the eventstream.

In this exercise, you'll ingest data from a sample data source that emits a stream of events related to observations of bicycle collection points in a bike-share system in which people can rent bikes within a city.

**Duration:** Approximately 30 minutes

**Note:** You need a Microsoft Fabric tenant to complete this exercise.

---

## Create a Workspace
Before working with data in Fabric, you need to create a workspace with the Fabric capacity enabled.

1. On the Microsoft Fabric home page at [https://app.fabric.microsoft.com/home?experience=fabric](https://app.fabric.microsoft.com/home?experience=fabric), select **Real-Time Intelligence**.
2. In the menu bar on the left, select **Workspaces** (the icon looks similar to 🗇).
3. Create a new workspace with the following configuration:
   - **Name**: `workspace46833824`
   - **Licensing mode**: **Trial** (Advanced section).
4. When your new workspace opens, it should be empty.

![image](https://github.com/user-attachments/assets/89000d51-941e-4832-aa6f-f74ff8f1c6f0)

![image](https://github.com/user-attachments/assets/66d75364-6309-4214-a90f-b4a4c2b88bef)



---

## Create an Eventhouse
Now that you have a workspace, you can start creating the Fabric items you'll need for your real-time intelligence solution. We'll start by creating an **Eventhouse**.

1. On the menu bar on the left, select **Home**.
2. In the **Real-Time Intelligence** home page, create a new **Eventhouse**.
   - Give it a **unique name** of your choice.
3. Close any tips or prompts that are displayed until you see your new empty eventhouse.

![image](https://github.com/user-attachments/assets/8bcfdafc-fd35-4ab2-af90-eebf30886e73)

![image](https://github.com/user-attachments/assets/4596f697-29fc-4c37-9cdc-8492a7086714)

![image](https://github.com/user-attachments/assets/e66366a8-afc6-414b-8aad-636b11c5cb15)




4. In the pane on the left, note that your eventhouse contains a **KQL database** with the same name as the eventhouse.
5. Select the **KQL database** to view it.
   - Currently, there are no tables in the database.

---

## Create an Eventstream
1. In the main page of your **KQL database**, select **Get data**.
2. For the data source, select **Eventstream > New eventstream**.
   - **Name the Eventstream**: `Bicycle-data`.
3. Your new eventstream will be created, and you'll be redirected to the **primary editor** to begin integrating sources.

![image](https://github.com/user-attachments/assets/d446e6e5-335d-4e96-b91c-482685439473)


---

## Add a Source
1. In the **Eventstream canvas**, select **Use sample data**.
2. **Name the source**: `Bicycles`, and select the **Bicycles sample data**.
   - Your stream will be mapped and displayed on the eventstream canvas.
  
![image](https://github.com/user-attachments/assets/f2e3b12e-a5c7-4a12-9052-657781ef5fc3)

![image](https://github.com/user-attachments/assets/4ee396eb-b793-45d2-8933-a616ab07c70c)



---

## Add a Destination
1. Use the **+ icon** to the right of the **Bicycle-data** node to add a new **Eventhouse** node.
2. Use the **pencil icon** in the new eventhouse node to edit it.
3. Configure the following setup options in the **Eventhouse pane**:
   - **Data ingestion mode**: Event processing before ingestion
   - **Destination name**: `bikes-table`
   - **Workspace**: Select the workspace you created earlier
   - **Eventhouse**: Select your eventhouse
   - **KQL database**: Select your KQL database
   - **Destination table**: Create a new table named `bikes`
   - **Input data format**: JSON

![image](https://github.com/user-attachments/assets/974d7f18-acd1-40ff-9d9e-a31663514eef)

![image](https://github.com/user-attachments/assets/30fb6236-a12e-4eb1-a14d-b4f76900ffe3)

![image](https://github.com/user-attachments/assets/9449d7e5-74a1-46c5-96df-cd100575cfe8)





4. In the **Eventhouse pane**, select **Save**.
5. On the toolbar, select **Publish**.
6. Wait for the data destination to become active.
7. Select the `bikes-table` node in the design canvas and view the **Data preview** pane to see the latest ingested data.

![image](https://github.com/user-attachments/assets/49abf320-a5b5-446a-866c-17d3f0a496f9)


---

## Query Captured Data
1. In the menu bar on the left, select your **KQL database**.
2. Refresh the view until you see the **bikes table**.
3. In the **... menu** for the `bikes` table, select **Query table > Records ingested in the last 24 hours**.

![image](https://github.com/user-attachments/assets/d632f9f9-127a-44c3-ba1f-9decf8923cb7)

![image](https://github.com/user-attachments/assets/de033eb9-90f0-4220-ad95-c0ec90bd7624)

![image](https://github.com/user-attachments/assets/e1fdc53e-896b-4b5b-8cb7-d730e8ea7a76)




The following query will be generated:

```kql
// See the most recent data - records ingested in the last 24 hours.
bikes
| where ingestion_time() between (now(-1d) .. now())
```
4. Run the query to see 100 rows of data from the table.



---

## Transform Event Data
1. In the menu bar on the left, select the `Bicycle-data` eventstream.
2. On the toolbar, select **Edit** to modify the eventstream.
3. In the **Transform events** menu, select **Group by** to add a new Group by node.
4. Connect the output of the `Bicycle-data` node to the input of the new **Group by** node.
5. Use the **pencil icon** to configure the **Group by** settings:
   - **Operation name**: `GroupByStreet`
   - **Aggregate type**: Sum
   - **Field**: `No_Bikes`
   - **Group aggregations by** (optional): `Street`
   - **Time window**: Tumbling
     - **Duration**: 5 seconds
     - **Offset**: 0 seconds

6. Save the configuration and return to the eventstream canvas.
7. Use the **+ icon** to the right of the **GroupByStreet** node to add a new **Eventhouse** node.
8. Configure the Eventhouse node with the following:
   - **Data ingestion mode**: Event processing before ingestion
   - **Destination name**: `bikes-by-street-table`
   - **Destination table**: Create a new table named `bikes-by-street`
   - **Input data format**: JSON

9. Select **Save** and **Publish** the changes.


![image](https://github.com/user-attachments/assets/4a6627b9-28d6-4c53-982b-40474a57ebad)

![image](https://github.com/user-attachments/assets/19189104-de24-4440-885a-bd02145dcaad)

![image](https://github.com/user-attachments/assets/2569ab92-5452-41fb-9562-3416746fecc2)

![image](https://github.com/user-attachments/assets/0992542b-1a5f-440c-a765-2828bea73a33)

Note that the trasformed data includes the grouping field you specified (Street), the aggregation you specified (SUM_no_Bikes), and a timestamp field indicating the end of the 5 second tumbling window in which the event occurred (Window_End_Time).

---

## Query the Transformed Data
1. In the menu bar on the left, select your **KQL database**.
2. Refresh the view until you see the `bikes-by-street` table.
3. In the **... menu** for the table, select **Query data > Show any 100 records**.
4. Modify the query to retrieve the total number of bikes per street within each 5-second window:

```kql
['bikes-by-street']
| summarize TotalBikes = sum(tolong(SUM_No_Bikes)) by Window_End_Time, Street
| sort by Window_End_Time desc, Street asc
```

5. Run the query.
   - The results will show the number of bikes observed in each street within each 5-second time period.

![image](https://github.com/user-attachments/assets/d526d2e4-794d-45e9-874b-a5dc1e3b43fb)

![image](https://github.com/user-attachments/assets/078f13d5-3db9-4a54-b590-88d9fb77513c)

![image](https://github.com/user-attachments/assets/9ccccf87-5b81-45c7-b2b1-a8d97ba96393)

![image](https://github.com/user-attachments/assets/a8f0f383-aa30-496b-8df8-22609d41f974)



---

