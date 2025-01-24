Running your queries requires the computing power of your BigQuery platform, which comes with a cost.

The cost of a query can vary significantly depending on how it’s written, the query parameters, and its complexity.

Complex queries can also take longer to process, leading to a poor experience when analysing data.



## In this exercise, we will cover:

- How to analyse the cost and resource requirements of a query
- How to optimise query writing to reduce costs and improve performance
- How to implement partitioning to further reduce costs and enhance performance


# 1. Cost and Bytes Billed

When you run a query, you can view important details in the Job Information tab:

- Duration of the query
- Bytes processed
- Bytes billed

![04-Cost-And-Partition-asset-1-Untitled](https://github.com/user-attachments/assets/5689896e-a0ea-4c72-a2fa-e9cf60e1f502)

![04-Cost-And-Partition-asset-1-Untitled](https://github.com/user-attachments/assets/7a823b62-6995-4b66-af24-611c8218d3bd)



1) Run the following query and check the Job Information tab for these key details:

- Bytes processed
- Bytes billed
- Duration

```
SELECT *
FROM `course17.gwz_sales_17`
WHERE 1=1
Note - The 1=1 condition is used to ensure this isn’t treated as a cached query, which would show 0 seconds and 0 bytes processed.
```
![04-Cost-And-Partition-asset-2-Untitled](https://github.com/user-attachments/assets/7712af5c-57e8-4cfe-b15f-dbc3030c08df)


2) Run the same query again. Check the Job Information tab.

What do you notice about the bytes processed, billed, and the duration?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

When you see zeros for all three fields, it means the result is stored in cache memory. BigQuery uses a temporary table to store the query results. So, when you run the query again, BigQuery doesn’t need to reprocess or recalculate the data.

![04-Cost-And-Partition-asset-3-Untitled](https://github.com/user-attachments/assets/ccda3884-7256-408c-b382-55ee6b7fd6a4)

If your query is cached and you can’t access the billed bytes, you can just make an unnecessary change in your query, for example change your WHERE condition from_ 1=1 to 2=2 which is still TRUE. The WHERE condition has changed, therefore, the query is not cached and BigQuery will process the data again.

![04-Cost-And-Partition-asset-4-Untitled](https://github.com/user-attachments/assets/5576d06e-7bb6-4e9b-b876-dd0175ec2eb4)


</details>

3) **Run the following query** and analyse the key information in the Job Information tab.

Are the bytes processed and billed the same? Can you explain why?

```
SELECT *
FROM `course17.gwz_ship_17`
WHERE 1=1
```

BigQuery applies a minimum billing threshold of 10MB for any query, regardless of its size. So, even if your query processes only 6MB, you’ll still be charged for 10MB. 💳

![04-Cost-And-Partition-asset-5-Untitled](https://github.com/user-attachments/assets/bafcc889-b9ff-4ba9-9a06-9e5af210086b)


4) Do the same with this query:

SELECT *
FROM `course17.gwz_mail_17`
WHERE 1=1

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The request is very small, BigQuery loads the minimum 10MB of bytes charged.

![04-Cost-And-Partition-asset-6-Untitled](https://github.com/user-attachments/assets/6a9c6b26-272d-48db-9aee-c5bc0e108c9f)


</details>


# 2. Column Selection

**REMINDER**

In Data Storage and the Data Platform, we learned that in BigQuery, data is stored in column-oriented tables.

![04-Cost-And-Partition-asset-7-Untitled](https://github.com/user-attachments/assets/fc8f94c2-58eb-4a9a-9ca5-f555e85174ee)

Columns are stored in different partitions and servers. So, if you select only specific columns, BigQuery doesn’t need to process the others, which reduces the cost and improves query performance.



Let’s test this with the gwz_sales_17 query:

```
SELECT *
FROM `course17.gwz_sales_17`
WHERE 1=1
```
We saw earlier that this query processed **145MB**.


**Detail of the cost of the request**.

Query results with gwz_sales_17

![04-Cost-And-Partition-asset-3-Untitled](https://github.com/user-attachments/assets/7dfb1cbf-27fd-45aa-b23d-150766c4022e)


1) Select only the products_id column and run the query. Check the Job Information tab. What do you notice about the bytes processed, billed, and the duration?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  products_id
FROM `course17.gwz_sales_17`
WHERE 1=1
```

The processed bytes are only 8.9MB and the duration 1s instead of 145MB and 5s

The processed bytes are only 8.9MB and the duration 1s instead of 145MB and 5s



</details>

2) Now select both orders_id and products_id columns and run the query. Check the Job Information tab.

What do you notice about the bytes processed, billed, and the duration?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  orders_id
  ,products_id
FROM `course17.gwz_sales_17`
WHERE 1=1
```
The processed bytes are 17.8MB and the duration is 2s. When you have 2 columns selected instead of one, you double the amount of data processed and the duration of the query.

![04-Cost-And-Partition-asset-10-Untitled](https://github.com/user-attachments/assets/001c3cf0-b5a6-424b-ba25-004c71f792a6)


</details>

3) Select the orders_id and products_id columns and filter by customers_id = 143176. Run the query and check the Job Information tab.

What do you notice about the bytes processed, billed, and the duration?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
  orders_id
  ,products_id
FROM `course17.gwz_sales_17`
WHERE 1=1
  AND customers_id = 143176
```

The bytes processed are 26.7 MB, which is three times more than when selecting a single column. Although the customers_id column isn’t in the SELECT statement, it is in the WHERE clause. BigQuery still needs to process the customers_id column to filter the relevant records.

The duration shows as 0 because most of the time is spent displaying the results in the user interface. In this case, there are only 65 rows to display since we filtered on a specific customer_id rather than over 1,100,000 rows. This results in a duration of 0, even though three columns were processed instead of one or two.

![04-Cost-And-Partition-asset-11-Untitled](https://github.com/user-attachments/assets/2b30aa4d-ca0f-4905-aea1-8bc40da1ce54)


</details>

# 3. Row Filtering and Partition

## 3.1 Row Filtering

In BigQuery, data is stored in a column-oriented format. If we select only a few columns, BigQuery should process and bill only for the selected columns, not the entire table.

**But what happens if we select just a specific row?**

Let’s test this with the gwz_sales_17 query:

```
SELECT *
FROM `course17.gwz_sales_17`
WHERE 1=1
```

Previously, this query processed **145MB**.


1) Now, select all columns but filter by date_date = "2021-08-01". Run the query and check the Job Information tab.

What do you notice about the bytes processed, billed, and the duration?

**Compare with gwz_sales_17.**

**<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The bytes processed are consistently 145MB because BigQuery partitions data by columns, not rows. Even when you select a limited number of rows, BigQuery must process all rows to gather data from the selected columns.

The duration shows as 0 because most of the time is spent displaying the results in the user interface. In this case, there are only 6,444 rows to display since we filtered on a specific date instead of over 1,100,000 rows. This results in a duration of 0, even though all columns were processed.

![04-Cost-And-Partition-asset-14-Untitled](https://github.com/user-attachments/assets/21fc5637-fffb-439e-93de-d6b5e2ea6a2d)

```
SELECT
  *
FROM `course17.gwz_sales_17`
WHERE date_date = "2021-08-01"
```

</details>**

## 3.2. Partitioning

It’s inefficient to scan the entire table when you only need data for a specific date. Fortunately, there’s a solution: Partitioning.

Partitioning allows you to organise rows based on a specific column, in addition to the default column-based storage. By filtering on the partitioned column, BigQuery only processes the relevant rows, not the entire dataset.

You can [partition by one specific column](https://cloud.google.com/bigquery/docs/partitioned-tables#date_timestamp_partitioned_tables), which can be:

- A time column (date, datetime, timestamp)
- An integer column
- The ingestion time in BigQuery (automatically calculated)

Partitioning is beneficial when you have a lot of rows per partition. If partitions are too small, it can slow down queries because BigQuery has to scan many partitions, increasing processing time.



1) Create an empty partition table called gwz_sales_17_partitioned and add all the data from the gwz_sales table to it. You can follow the steps in this [video tutorial](https://drive.google.com/file/d/16EZD-_vO9AZmgBf24r9KVjA_4D8xHvpF/view?usp=sharing) 

- Create an empty table with one column and partition it on that column.
- Run a query with the overwrite option to save the result in your partitioned table.


2) Select all columns from gwz_sales_17_partitioned and filter by date_date = "2021-08-01".

Compare the bytes processed and the duration with the previous query.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

With the partition table, BigQuery processed much less data than without partitioning.

![04-Cost-And-Partition-asset-16-Untitled](https://github.com/user-attachments/assets/0860c5fb-38aa-46ec-a23a-c0525bd1f28b)

</details>


# 4. Billing by “Volume”, Not Complexity


BigQuery charges based on the volume of data processed, not the complexity of the queries or the resources required to run them.

1) Run the following query. Check the Job Information tab for details on bytes processed, billed, and the duration.

```
 SELECT *
 FROM `course17.gwz_campaign_17` t1
 WHERE 2=2
```

Also, go to the **Execution Details** tab to check the **Slot time consumed**.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

  Duration - 0 sec Bytes processed - 370KB (very few) Slot time consumed - 42ms Bigquery can parallelize your calculation on several servers. The slot time consumed represents the equivalent in time of your calculation on a single server.

![04-Cost-And-Partition-asset-18-Untitled](https://github.com/user-attachments/assets/6eab3cf1-cf0a-4913-b710-2fc6b49daa3c)

![04-Cost-And-Partition-asset-19-Untitled](https://github.com/user-attachments/assets/4f9a2352-8820-48d9-aecf-697fbb19f608)


</details>


2) Run this query next. It may take a while to complete.

```
SELECT *
FROM `course17.gwz_campaign_17` t1
INNER JOIN `course17.gwz_campaign_17` t2 ON t1.campaign_name=t1.campaign_name
```

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

We made a mistake in our query by using the condition t1.campaign_name = t1.campaign_name, which is always TRUE. This caused a CROSS JOIN, leading to **59,259,204** rows instead of the expected **7,698 rows**, resulting in unnecessary and excessive processing.

Because of this error, BigQuery had to perform a very complex calculation, which took a lot of resources. The slot time consumed increased significantly to **4 minutes 35 seconds** instead of the expected **42 milliseconds**. Despite this, the amount of data processed remained the same, so the cost didn’t change.

**Summary:**

- Duration: 19 seconds
- Bytes processed: 370KB (minimal)
- Slot time consumed: 4 minutes 35 seconds

BigQuery parallelises calculations across multiple servers, and the slot time consumed shows how long the calculation would have taken if it had been done on just one server.

![04-Cost-And-Partition-asset-20-Untitled](https://github.com/user-attachments/assets/5e276640-5340-4eb1-8eb4-fe5cfd40c062)

![04-Cost-And-Partition-asset-21-Untitled](https://github.com/user-attachments/assets/7c8c071e-e40d-4da7-af10-7b42009e9f93)


</details>

# Bonus

## Join Request

Understanding the cost and performance during a join operation can be tricky. Let’s break it down.

### Cost when Join

1) Analyse Bytes Processed:

- Select all columns from gwz_orders_17 and check how many bytes are processed.
- Select orders_id, ship_cost, and log_cost from gwz_ship_17 gwz_ship_17 and check the bytes processed.
- Perform a join between gwz_orders_17 and gwz_ship_17 to add log_cost and ship_cost to orders. Run the query and analyse the number of bytes processed.

**What do you observe?**

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

In the join, you add the processed bytes from the two source tables.

![04-Cost-And-Partition-asset-22-Untitled](https://github.com/user-attachments/assets/83364c23-f5ee-4d53-a651-1c2500010b90)

![04-Cost-And-Partition-asset-23-Untitled](https://github.com/user-attachments/assets/c9e95010-6779-4c90-bfec-bd943049b58f)


</details>


### Request Execution Details

After running a query, you can access key information in the Job Information tab, like processed bytes, billed bytes, and duration. But for more insights, check the Execution Details tab.

Check the Execution Details

<details>
    <summary> <font color="red"><b>Check</b></font></summary>

#### Parallelisation Ratio

![04-Cost-And-Partition-asset-7-Untitled](https://github.com/user-attachments/assets/bbabbbe8-ef3f-4b9d-94ef-d750f1d2148f)

Here, you can see the ratio of consumed slot time to elapsed time, indicating how well the task was parallelised across servers.

#### Calculation Stages

The tab breaks down the steps of the calculation. In this case, it’s straightforward, but with more complex queries (like joins, aggregations, etc.), this tab provides detailed insights.

For a more complex query, try this one:

```
   SELECT
     ### Key ###
     o1.orders_id
     ###########
     ,COUNT(o2.orders_id) AS orders_number
   FROM `course17.gwz_orders_17` o1
   LEFT JOIN `course17.gwz_orders_17` o2 ON o1.customers_id = o2.customers_id AND o1.orders_id>=o2.orders_id
   GROUP BY o1.orders_id
```

![04-Cost-And-Partition-asset-9-Untitled](https://github.com/user-attachments/assets/fb404e86-d531-42e6-b123-74a579d98542)


</details>


Feel free to explore the Execution Details during exercises to understand how your queries are processed.


## Optional

Redo the exercise from the beginning but focus on analysing the Execution Details tab instead of the Job Information tab.



## Clustering

[Read the documentation on clustering](https://cloud.google.com/bigquery/docs/clustered-tables)

(Optional) Test clustering by creating a new table gwz_sales_clustered.
