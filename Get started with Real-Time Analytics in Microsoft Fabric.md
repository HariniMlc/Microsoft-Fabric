# Get Started with Real-Time Intelligence in Microsoft Fabric

## Overview
Microsoft Fabric provides Real-Time Intelligence, enabling you to create analytical solutions for real-time data streams. In this exercise, you'll use the Real-Time Intelligence capabilities in Microsoft Fabric to ingest, analyze, and visualize a real-time stream of stock market data.

**Lab Duration**: Approximately 30 minutes

**Note**: You need a Microsoft Fabric tenant to complete this exercise.

## Steps

### Create a Workspace
1. Go to [Microsoft Fabric](https://app.fabric.microsoft.com/home?experience=fabric).
2. Select **Real-Time Intelligence**.
3. In the left menu, select **Workspaces** (🗇 icon).
4. Create a new workspace named `workspace46828824`, with licensing mode set to **Trial**.
5. Your new workspace should open empty.

   ![image](https://github.com/user-attachments/assets/198cab85-080e-4809-91f4-c9ad0e8768cd)


   ![image](https://github.com/user-attachments/assets/48536b22-5c2d-493c-a099-1200875a9973)


   ![image](https://github.com/user-attachments/assets/ce3b8a5b-a073-45ed-b52c-94b9cdb8143a)



### Create an Eventstream
1. Select the **Real-Time hub** in the left menu.
2. In the **Connect to** section, select **Data sources**.
3. Find the **Stock market sample data source** and select **Connect**.
4. Name the source `stock` and the eventstream `stock-data`.

   ![image](https://github.com/user-attachments/assets/d5d25528-f199-400c-861b-a8208bdde6f0)

   ![image](https://github.com/user-attachments/assets/395fd1ed-51ad-4826-a3e6-e6850191e28d)

   ![image](https://github.com/user-attachments/assets/d3e914ac-795c-4b3e-8654-b4cf87c8c042)

   ![image](https://github.com/user-attachments/assets/1d77b856-a847-402a-85ca-e77576e28fed)

   ![image](https://github.com/user-attachments/assets/9d4d6bb8-0560-497d-976a-4a066253db4a)




6. Select **Next** and wait for creation, then select **Open eventstream**.

### Create an Eventhouse
1. Go to **Home** and create a new Eventhouse.
2. In the left pane, note your Eventhouse contains a KQL database with the same name.
3. Select the database and note the associated queryset.
4. Select **Get data** on the KQL database main page.
5. For the data source, select **Eventstream > Existing eventstream**.
6. Create a new table named `stock`.
7. Select your workspace and the `stock-data` eventstream. Name the connection `stock-data`.

   ![image](https://github.com/user-attachments/assets/25e21554-b81c-490e-8ce7-b82b86590841)

   ![image](https://github.com/user-attachments/assets/636daffa-fe71-4e4a-9902-e02f62257638)

   ![image](https://github.com/user-attachments/assets/cbb59a51-d0be-4bf0-b780-f0b41867662d)

   ![image](https://github.com/user-attachments/assets/4456eebd-44a5-43ea-8963-16227c0fe3fd)

   ![image](https://github.com/user-attachments/assets/dd25fd81-e52b-4a55-8d06-4e10827d4c4a)

   ![image](https://github.com/user-attachments/assets/cd230630-b376-452e-bd66-cb871a5ca4c3)

   ![image](https://github.com/user-attachments/assets/cb284368-b097-4539-a09d-85494873a574)

   ![image](https://github.com/user-attachments/assets/ebc537fb-fa88-4b44-ba2e-cba55e9a50fb)

   ![image](https://github.com/user-attachments/assets/b1b155a0-98c5-4e5d-b76b-c9b7f01d26fa)

   ![image](https://github.com/user-attachments/assets/3b3f9e67-a34a-4556-936a-6d8d5b7e759a)

   ![image](https://github.com/user-attachments/assets/daab5f9f-c7f3-4bed-8107-191b996bf733)




### Verify Eventstream
1. In the **Real-Time hub**, view the **My data streams** page.
2. The stock table and `stock-data-stream` should be listed.
3. Select **Open eventstream**.

Tip: Select the destination on the design canvas, and if no data preview is shown beneath it, select Refresh.

In this exercise, you've created a very simple eventstream that captures real-time data and loads it into a table. In a real soltuion, you'd typically add transformations to aggregate the data over temporal windows (for example, to capture the average price of each stock over five-minute periods).

Now let's explore how you can query and analyze the captured data.



### Query the Captured Data
1. Select your eventhouse database in the left menu.
2. Select the queryset for your database.
3. Run the following KQL query to see 100 rows of data from the table:
   ```kql
   stock
   | take 100
   ```
4. Modify the query to retrieve the average price for each stock symbol in the last 5 minutes:

  ```kql
  stock
  | where ["time"] > ago(5m)
  | summarize avgPrice = avg(todecimal(bidPrice)) by symbol
  | project symbol, avgPrice
  ```

5. Highlight the modified query and run it to see the results.

6. Wait a few seconds and run it again, noting that the average prices change as new data is added to the table from the real-time stream.

   ![image](https://github.com/user-attachments/assets/07d543bb-9e8d-4096-8d87-42468c293d4b)

   ![image](https://github.com/user-attachments/assets/2ace5379-4123-49ce-bc7c-b7547b2201c7)

   ![image](https://github.com/user-attachments/assets/e33cfef6-558f-4892-8cd2-7e36e32c8878)

   ![image](https://github.com/user-attachments/assets/1c5ca2dd-38ea-40f6-b8c6-eedb8d245a4f)

   ![image](https://github.com/user-attachments/assets/45b72730-fb3d-41ef-95ca-557cfc989afc)






### Create a Real-Time Dashboard
1. Pin the KQL query to a new dashboard with the following settings:
    Dashboard name: Stock Dashboard
    Tile name: Average Prices
    Change the visual from Table to Column chart.
   
3. At the top of the dashboard, switch from Viewing mode to Editing mode.
4. Select the Edit (pencil) icon for the Average Prices tile.
5. In the Visual formatting pane, change the Visual from Table to Column chart:
6. At the top of the dashboard, select Apply changes and view your modified dashboard.

Now you have a live visualization of your real-time stock data.


![image](https://github.com/user-attachments/assets/9fde3258-7518-44ad-9c0b-dacde0276f0a)

![image](https://github.com/user-attachments/assets/dffc24fd-3868-4cfd-b0dc-5e19ee0e8bdd)

![image](https://github.com/user-attachments/assets/1c89e16a-e463-45fa-b107-83833f00dd05)

![image](https://github.com/user-attachments/assets/68312254-2e67-4b42-b4b5-a75bfbc7ba62)

![image](https://github.com/user-attachments/assets/161cafa0-6a8d-4c68-a56f-e63fcca60e6e)





### Create an Alert
1. In the dashboard, select Set alert.
2. Create an alert with the following settings:

Run query every: 5 minutes
Check: On each event grouped by
Grouping field: symbol
When: avgPrice
Condition: Increases by
Value: 100
Action: Send me an email
Save location:
Workspace: Your workspace
Item: Create a new item
New item name: A unique name of your choice

3. Create the alert and wait for it to be saved. Then close the pane confirming it has been created.
4. In the menu bar on the left, select the page for your workspace (saving any unsaved changes to your dashboard if prompted).
5. On the workspace page, view the items you have created in this exercise, including the activator for your alert.
6. Open the activator, and in its page, under the avgPrice node, select the unique identifier for your alert. Then view its History tab.
7. Your alert may not have been triggered, in which case the history will contain no data. If the average stock price ever changes by more than 100, the activator will send you an email and the alert will be recorded in the history.

![image](https://github.com/user-attachments/assets/442e55f6-54f3-4ffa-a2a3-561433eb2fed)

![image](https://github.com/user-attachments/assets/64238f80-9733-4258-9fba-db032a86a202)

![image](https://github.com/user-attachments/assets/03efe367-6d07-45be-b4fa-9244d491b42d)

![image](https://github.com/user-attachments/assets/0e71478c-e6f9-4871-8a9e-c6c7309d034b)

![image](https://github.com/user-attachments/assets/22954139-c63b-47fd-ab42-9f4ad573e257)


![image](https://github.com/user-attachments/assets/62d7c67f-b9f8-4775-84cc-1333a2539508)




### Clean Up Resources
Delete the workspace created for this exercise.
Select Workspace settings and Remove this workspace.

Congratulations
You have successfully completed this lab. Click End to mark the lab as Complete.






   
