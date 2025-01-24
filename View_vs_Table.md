## Objective

The objective of this exercise is to understand the difference between views and tables.


In this challenge, you will:

- Create your first views and tables
- Understand when to choose each method based on the situation
- Simplify your data pipeline by combining multiple transformations into a single view


Steps:
1)  Save Your Queries

Learn the importance of saving transformation queries in views, enabling you to restart your data pipelines in the future.

2) View-Only Method

Explore the benefits of the view-only method, particularly for regular updates and small data volumes.

3) View + Table Method

Discover the advantages of the View + Table method, especially for handling large data volumes with infrequent updates.

## Context

In your previous work on Circle, you calculated valuable KPIs on stock and shortages for the purchasing team. They loved the metrics you provided and want to use them regularly.

But there’s a challenge — stock levels change rapidly. Every hour, we receive an update from the warehouse with the latest stock data, and the purchasing team needs the most up-to-date numbers. We’ve just received the latest stock sheet in this Google sheet, which includes:

- A few orders placed by the purchasing team that increased the forecasted stock.
- Deliveries of clothing items, including a large shipment of “Brassière” (🇬🇧: Bra) and “Crop-tops.”
- Deliveries of accessories, such as “Gourdes” (🇬🇧: Flasks).
- Sales that led to some products becoming out of stock.



# 1. Saving your Request

## 1.1. Views on BigQuery

We need you to update the KPIs calculated last year with this new data.

1. How will you manage this? What queries and transformations should you apply?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>
1. Replace the Data: Start by updating the data in your connected Google Sheet with the new values.

2. Re-run Queries: After updating the sheet, you’ll need to re-run the queries that calculate circle_stock_name, followed by circle_stock_cat, and then circle_stock_kpi. Additionally, don’t forget to re-run the aggregate query that calculates stock statistics by model type from circle_stock_kpi.

</details>

We performed several transformations from our raw circle_stock table to our clean, enriched circle_stock_kpi table (excluding circle_stock_kpi_top for this exercise).


![02-View-And-Table-Circle-Inventory-Management-asset-1-Untitled](https://github.com/user-attachments/assets/6d7fa4ac-4a9b-4d93-b469-0c2f692f79d0)

Lineage of circle_stock

It’s essential to save requests 1, 2, and 3 as views. This way, if we need to perform the transformation again, we won’t have to rewrite them from scratch.



2) Go back to course14 and retrieve the query used to create circle_stock_name from circle_stock.

Save it as a view and name it circle_stock_name_view.

[Documentation](https://cloud.google.com/bigquery/docs/views#view_naming) to create a view on BigQuery from the console

![02-View-And-Table-Circle-Inventory-Management-asset-2-Untitled](https://github.com/user-attachments/assets/851fa8a5-1662-41e7-a908-d0e0713dc3e9)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  ### Key ###
  CONCAT(model,"_",color,"_",IFNULL(size,"no-size")) AS product_id
  ###########
  ,model
  ,color
  ,size
  -- name
  ,model_name
  ,color_name
  ,CONCAT(model_name," ",color_name,IF(size IS NULL,"",CONCAT(" - Taille ",size))) AS product_name
  -- product info --
  ,t.new AS product_new
  ,price
  -- stock metrics --
  ,forecast_stock
  ,stock
FROM `course15.circle_stock` AS t
WHERE TRUE
ORDER BY product_id
```

</details>


3) Go back to course15 and retrieve the query used to create circle_stock_cat from circle_stock_name. Save it as a view and name it circle_stock_cat_view.
Save it as a view and name it circle_stock_cat_view.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  ### Key ###
  product_id
  ###########
  ,model
  ,color
  ,size
  -- category
  ,CASE
    WHEN REGEXP_CONTAINS(LOWER(model_name),'t-shirt') THEN 'T-shirt'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'short') THEN 'Short'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'legging') THEN 'Legging'
    WHEN REGEXP_CONTAINS(LOWER(REPLACE(model_name,"è","e")),'brassiere|crop-top') THEN 'Crop-top'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'débardeur|haut') THEN 'Top'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'tour de cou|tapis|gourde') THEN 'Accessories'
    ELSE NULL
  END AS model_type
  -- name
  ,model_name
  ,color_name
  ,product_name
  -- product info --
  ,product_new
  ,price
  -- stock metrics --
  ,forecast_stock
  ,stock
FROM `course15.circle_stock_name`
WHERE TRUE
ORDER BY product_id
```


</details>


4) Go back to course15 and retrieve the query used to create circle_stock_kpi from circle_stock_cat. If necessary, update the query to include the latest data. Save it as a view and name it circle_stock_kpi_view. Ignore the **top_products** column.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  ### Key ###
  product_id
  ###########
  ,model
  ,color
  ,size
  -- category
  ,model_type
  -- name
  ,model_name
  ,color_name
  ,product_name
  -- product info --
  ,product_new
  -- stock metrics --
  ,forecast_stock
  ,stock
  ,IF(stock>0,1,0) AS in_stock
  -- value
  ,price
  ,ROUND(stock*price,2) AS stock_value
FROM `course15.circle_stock_cat`
WHERE TRUE
ORDER BY product_id
```


</details>

## 1.2. Data pipeline


Now let’s update the data pipelines so that the KPIs are updated with the latest values.


1) Copy the data from the new updated stock sheet and paste it into your own 15 - circle_stock sheet.


2) Run the circle_stock_kpi_view. Is the data updated in the query results?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The data is not up to date. You can check this by looking at the first row of the table with the product_id “ACC001-B01-BL-L_BL_L” which has a stock of 50 in the new updated stock sheet.

This is normal. circle_stock_kpi_view asks for data from the circle_stock_cat tables that have not been updated.


</details>

3) Now, run successively the following views:

- circle_stock_name_view and overwrite circle_stock_name with the query result.
- circle_stock_cat_view and overwrite circle_stock_cat with the query result.
- circle_stock_kpi_view and overwrite circle_stock_kpi with the query result.



4) Select the circle_stock_kpi table. Is the data updated?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The data is now updated with the new sheet.

To verify this, check the first row of the table for the product_id “ACC001-B01-BL-L_BL_L”. The stock value should now reflect 50, matching the new data in the updated stock sheet.

</details>

# 2. ‘View only’ Method

## 2.1. Creating View

You’ve run your first data pipelines, but the circle_stock table is updated every hour with new data from the warehouse.


1) How long did it take you to update the data pipeline? Would you be willing to do it every hour?

**View only** method

To avoid recalculating everything hourly, we can save our transformations in views instead of intermediate tables that need constant updating.



We don’t save the intermediate results in a table; instead, we use a view. When you call a view, it runs the query again from the source tables, ensuring your data is always up-to-date.



We will delete the intermediate tables circle_stock_name, circle_stock_cat, and circle_stock_kpi and replace them with views.



2) For circle_stock_name, perform the following operations:

- Delete the circle_stock_name table.
- Modify circle_stock_name_view and rename it to circle_stock_name.
- Delete circle_stock_name_view.


Note - Editing/Renaming a view

![02-View-And-Table-Circle-Inventory-Management-asset-5-Untitled](https://github.com/user-attachments/assets/14481909-eb54-4b8a-a441-ce6eec3ae6dd)

Open and edit the view in details tab

Save the view with the right option

- Use “Save view” if you update the view without changing its name.
- Use “Save view as“ if you need to save it as a new view with a different name.

![02-View-And-Table-Circle-Inventory-Management-asset-6-Untitled](https://github.com/user-attachments/assets/9a8d0e4f-ed2e-46f1-9d22-3171f5efd165)


3) Repeat the same steps for circle_stock_cat and circle_stock_kpi.



4) Make a small change to your stock sheet.

E.g. change the stock and forecasted stock for product_id ACC001-B01-BL-L from 50 to 49.
- Run the circle_stock_kpi view and verify that the data is up-to-date.



## 2.2. Optimizing your Data Pipeline

Now, by simply running circle_stock_kpi, you can access the transformed table with:

- Enriched columns: product_id, product_name, model_type, in_stock, stock_value (excluding top_products).
- Automatically updated data directly from the circle_stock Google Sheet without needing to manually update the tables.



However, you need 3 consecutive views to progressively add:

- product_id + product_name → model_type → in_stock + stock_value
- circle_stock_name → circle_stock_cat → circle_stock_kpi

This can become messy with so many views for a simple transformation. But don’t worry—there’s a solution!



Combine these three views into one. Since the calculations and transformations are independent, they can be done simultaneously (instead of sequentially). Save the result as cc_stock.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

By combining all your transformations into a single query, you make the process cleaner and easier to understand. Plus, by saving this query as a view, you ensure that each time you run it, you’re working with the most up-to-date raw data.

```
SELECT
  ### Key ###
  CONCAT(model,"_",color,"_",IFNULL(size,"no-size")) AS product_id
  ###########
  ,model
  ,color
  ,size
  -- category
  ,CASE
    WHEN REGEXP_CONTAINS(LOWER(model_name),'t-shirt') THEN 'T-shirt'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'short') THEN 'Short'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'legging') THEN 'Legging'
    WHEN REGEXP_CONTAINS(LOWER(REPLACE(model_name,"è","e")),'brassiere|crop-top') THEN 'Crop-top'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'débardeur|haut') THEN 'Top'
    WHEN REGEXP_CONTAINS(LOWER(model_name),'tour de cou|tapis|gourde') THEN 'Accessories'
    ELSE NULL
  END AS model_type
  -- name
  ,model_name
  ,color_name
  ,CONCAT(model_name," ",color_name,IF(size IS NULL,"",CONCAT(" - Taille ",size))) AS product_name
  -- product info --
  ,t.new AS product_new
  -- stock metrics --
  ,forecast_stock
  ,stock
  ,IF(stock>0,1,0) AS in_stock
  -- value
  ,price
  ,ROUND(stock*price,2) AS stock_value
FROM `course15.circle_stock` AS t
WHERE TRUE
ORDER BY product_id
```

</details>


The purchasing team needs an updated view of the aggregated stock and shortage KPIs by model_type. Can you create this and save the result as cc_stock_model_type 💾? Then, compare the result with the previous stock. What can you observe?

Note: To easily compare two queries in BigQuery, you can split the screen, placing one query in the right pane and the other in the left pane. - Split tabs documentation



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  model_type
  ,COUNT(in_stock) AS nb_products
  ,SUM(in_stock) AS nb_products_in_stock
  ,ROUND(AVG(1-in_stock),3) AS shortage_rate
  ,SUM(stock_value) AS stock_value
FROM `course15.cc_stock`
GROUP BY model_type
ORDER BY model_type
Previous stock view

# Previous Stock view

SELECT
  model_type
  ,COUNT(in_stock) AS nb_products
  ,SUM(in_stock) AS nb_products_in_stock
  ,ROUND(AVG(1-in_stock),3) AS shortage_rate
  ,SUM(stock_value) AS stock_value
FROM `course15.circle_stock_kpi`
GROUP BY model_type
ORDER BY model_type
```
![02-View-And-Table-Circle-Inventory-Management-asset-7-Untitled](https://github.com/user-attachments/assets/b3dd9f6e-7cae-49e8-b75f-364f638df18c)


</details>

# 3. Table + View Method

## 3.1. Creating Views for Circle Stock and Sales Data

You’ve created views to process the circle_stock data:

- cc_stock - enriches raw data with new columns.
- cc_stock_model_type - aggregates stock KPIs by model_type.
Now, let’s save our queries for circle_sales as views too!



1) Return to the SOLUTION for exercise A (Challenge 2 of Aggregation, Date & Time Functions, Testing lecture).

2) Retrieve the query to create circle_sales_daily from circle_sales. Save it as a view named cc_sales_daily_view.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  product_id
  ,SUM(qty) AS qty_91
  ,ROUND(SUM(qty)/91,2) AS avg_daily_qty_91
FROM `course15.circle_sales`
WHERE TRUE
  AND date_date >= DATE_SUB('2021-10-01',INTERVAL 91 DAY)
GROUP BY product_id
```

</details>


3.2. Compare Requests Performance
Run cc_sales_daily_view and cc_stock, then compare their execution performance — **number of records read** and **computing time**. How many rows are in the source tables circle_stock and circle_sales?

Note - How to check the detailed performance of a query execution?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>


![02-View-And-Table-Circle-Inventory-Management-asset-9-Untitled](https://github.com/user-attachments/assets/0303f1ae-e2dc-4056-82a9-942678c197e1)

Processing sales data takes significantly longer because the sales source data sheet has over 20,000 rows, compared to just 450 rows in the stock sheet.
Circle is a young company with relatively few sales at the moment, focusing on a 4-month period. However, if you were dealing with an older company like Greenweez over a 2-year period, you’d be handling several million rows, which would take considerable time to process.



</details>

**Saving Query Results in a Table**

When dealing with large queries on extensive datasets, it can be helpful to save the query result in a table. This way, if you need to access a specific row, you can work with the smaller, aggregated table instead of recalculating everything from the raw data. This is especially useful when having the latest data isn’t crucial, meaning you don’t need to update your table every hour.

**Benefits of Storing Sales Data**

The sales data is updated only once a day and has many more rows than the stock data. Therefore, storing the results of cc_sales_daily_view in a table is beneficial. This avoids re-aggregating the data every time you want to access the sales statistics of a specific product.



2) Run cc_sales_daily_view and save the result as the cc_sales_daily table.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  *
FROM `course15.cc_sales_daily_view`
```


</details>


To avoid manually updating the data daily, we’ll explore orchestration techniques in upcoming courses to automate the data pipeline updates.



# Summary

## 1 - Saving Your Queries as Views

When source data is updated, your query needs to be re-executed . Therefore, it’s crucial to save your queries as views.

## 2 - Views Up-to-Date Data

If your source data is regularly updated and you need fresh data for analysis, it’s best to use views instead of storing results in a table. This ensures you always get the most up-to-date data when running the query. This approach is particularly useful when the source data is small and calculations are quick and easy.

## 3 - Tables Large, Time-Consuming Calculations

When dealing with large datasets and complex, time-consuming calculations, it’s helpful to store the results of your view queries in a table. This allows you to access the results quickly without recalculating everything from the source data. This is especially useful when having the very latest data isn’t critical.

## 4 - Naming Conventions 

When saving query results in a table, use the _view suffix for your view, and name the table without the suffix. This naming convention makes it clear which is the view and which is the table.

When using only a view without storing results in a table, omit the _view suffix to indicate that the query directly provides the view’s result.



In future tasks, consider whether it’s more appropriate to use a view or a table, depending on the query type.

During development, it’s often easier to work with views to avoid refreshing the table with every transformation.

Once development is complete, you can update the structure and create the table as needed.
