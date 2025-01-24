#### This challenge builds on Challenge 3 of the Joins & Testing Respository. Feel free to review that exercise before starting this one.

To generate our financial reports, we have aggregated multiple data sources and performed many transformations. It can be challenging to see how each step **connects**.

However, having a clear overview is crucial for:

- Orchestration - Restarting the data flow to ensure up-to-date analysis when new information is added.
- Maintenance - Identifying bugs, assessing their impact, fixing them, and restarting the data pipeline.
- Evolution - Finding opportunities to improve and enrich the data pipeline to meet new business needs.



## Objective
- Understand the importance of data lineage.
- Create your first data lineage visualisation.
- Create your first SQL orchestration.



## Steps 

1. Determine the **data lineage**  of our financial transformation pipeline to gain insight and understanding.

2. Use **scheduled queries** to automate the orchestration and execution of our data pipeline.



# 1. Data Lineage

## 1.1 How to Create a Diagram

To make the diagrams you can use: https://app.diagrams.net/

How to use it [Five Features That Help You to Draw Better Diagrams](https://medium.com/@semeltheone_87908/five-features-which-help-you-to-draw-better-diagrams-289bd6537fb0)



## 1.2 DAG - Create Data Lineage

1) List the sequential data sources, tables, and views needed to move from your 3 data sources (gwz_sales, gwz_product, and gwz_ship) to your financial day report (gwz_finance_day).

Remember: When view A relies on another table or view B, A is dependent on B.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gwz_sales
gwz_product
gwz_sales_margin
gwz_orders
gwz_ship
gwz_orders_operational
gwz_finance_day

</details>

When a view A calls another table/view B, we say that there is a dependency from A to B.



2) Add all **dependencies** to the previous view listing. This list will help you understand the relationships between your data sources and views.

This list is not very clear or visual at the moment. To better understand the data lineage, we’ll create a graph known as a Directed Acyclic Graph (DAG).

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gwz_sales
gwz_product
gwz_sales_margin
gwz_sales
gwz_product
gwz_orders
gwz_sales_margin
gwz_ship
gwz_orders_operational
gwz_orders
gwz_ship
gwz_finance_day
gwz_orders_operational


</details>

3) Complete the following DAG with the correct view/table names.

DAG of GWZ Finance

![03-Gwz-Finance-Data-Lineage-With-DAG-And-Orchestration-asset-1-Untitled](https://github.com/user-attachments/assets/e79a0629-b8f7-41cb-a9c7-dfcb4e13748f)




# 2. Data Orchestration

You’ve understood the data lineage and the various transformations. Now, we can move on to orchestration🪄.

## 2.0. RECAP - View-only vs View+Table

In challenge 2 of Joins and Testing course, we saved queries as tables.

During development, using the view-only method is easier and saves time when updating and evolving views, as you don’t need to recalculate the tables each time.

However, once the data pipeline development is complete, it’s important to decide whether it’s better to continue using views alone or to incorporate tables for efficiency.



## 2.1. gwz_sales_margin& gwz_finance_day

1) How many rows are there in the source table?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

  There are more than 1,300,000 rows in the source table gwz_sales_margin. In the execution details tab, we see that some steps of the calculation take 1 or more seconds. The queries use a lot of data.

</details>

2) Run gwz_sales_margin and look at the run (execution) details tab. The company needs a daily financial report. How often should you update the data?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

In addition, the financial tracking report only needs to be updated once a day, so freshness is not essential. Therefore, it would be better to store gwz_sales_margin with the table method rather than with the view only.


</details>

3) Would you prefer to use the view-only method or the table method to save the output?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

In addition, the financial tracking report only needs to be updated once a day, so freshness is not essential. Therefore, it would be better to store gwz_sales_margin with the table method rather than with the view only.

gwz_finance_day calls several views in succession. Furthermore, this will be the table used and requested by the finance team several times a day for their analysis. It therefore seems logical to store the final result in a table rather than in a view only to avoid recalculating each time and improve the performance of the queries to obtain the data.


</details>

**Development Phase**

4) Imagine you’re in the development process. Run gwz_sales_margin AND gwz_finance_day using the view-only method.



**Validation Phase**

5) Now let’s imagine we validated the code. We are now ready to change gwz_sales_margin and gwz_finance_day from view-only backup method to table saving method.



6) Switching the Saving Method

- Rename the two views with the suffix _view.
- Delete the old views without the suffix.
- Run the query and save the result in the gwz_sales_margin and gwz_finance_day tables.

Note - Editing/Renaming a view

# 3. Transformation Orchestration

Here is the DAG of the data pipeline

- The tables have a RED outline.
- The source data tables, shown in BLUE (gwz_sales, gwz_product, gwz_ship), are updated every morning at 6:50 a.m.


DAG of the data pipeline - GWZ Finance

![03-Gwz-Finance-Data-Lineage-With-DAG-And-Orchestration-asset-5-Untitled](https://github.com/user-attachments/assets/2c683ccf-d58b-4d6e-a846-e881e59d7b27)


We’ve decided to use the table-saving method for gwz_sales_margin and gwz_finance_day to reduce computational costs and boost performanc.

➡ But these tables won’t update automatically. 

To keep the finance team updated, you’d need to log in every morning at 7 a.m. to recalculate the results. But waking up early for manual updates isn’t ideal, right? 

Good news!  We have a solution — **Scheduled Query** will handle the updates for you!



## 3.1. Scheduled Query

Create a scheduled query named gwz_sales_margin_update to run gwz_sales_margin_view daily at 7 a.m. This query should overwrite the gwz_sales_margin table with updated data. **Set today at 7 a.m. as the start date.**

```
SELECT * FROM `course16.gwz_sales_margin_view`
```

Help - [Create a scheduled query documentation](https://cloud.google.com/bigquery/docs/scheduling-queries#set_up_scheduled_queries)



Go to the scheduled query tabto view your new scheduled query. Trigger an immediate manual run by clicking on Scheduled backfill and selecting Run a one-time transfer.



Do the same for gwz_finance_day but at 7:10am to be sure that the update of gwz_sales is finished.

Help - Create a scheduled query documentation and Scheduled backfill documentation



## 3.2 - Test Scheduled Query with a New Data Update

**Data Import**

1) New files with additional data from October 1, 2021, have been added. We need to update the source data for gwz_sales and gwz_ship by overwriting them with the new files.

- Delete the gwz_sales table in your project. Then, open the gwz_update_sales table and copy it to your course16 dataset as gwz_sales.
- Delete the gwz_ship table in your project. Then, open the gwz_update_ship table and copy it to your course16 dataset as gwz_ship.



2) Manually trigger your two scheduled queries in sequence:
- First, update gwz_sales_margin_update
- After it completes (wait a few minutes), execute gwz_finance_day_update with the new data from October 1, 2021.
- Check if everything worked correctly by selecting the rows from the gwz_finance_day table and ordering them by date_date in DESC order.

Help: Refer to the [Scheduled backfill documentation](https://cloud.google.com/bigquery/docs/scheduling-queries#set_up_a_manual_run_on_historical_dates)



!!Pause your scheduled query !! (Please make sure to pause your scheduled query to avoid fees!)



Bonus
Test Orchestration
We’ve automatically executed the table updates, but we didn’t run the tests. Errors could exist in the source data or from updates made to the views since the last test.

Since you’d rather enjoy your evenings than run manual queries early in the morning, we’ll use Scheduled Query to automate the tests.



List all tests related to our dataflow tables.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gwz_sales_pk
gwz_product_pk
gwz_sales_margin_pk
gwz_sales_margin_column-purchase_price
gwz_orders_pk
gwz_ship_pk
gwz_orders_operational_pk
gwz_orders_operational_conservation-ship1 and ship2
gwz_orders_operational_column-ship


</details>

2) **Save the query** as dataflow_finance_testing_view in the course16_test dataset. Run it and save the result in the dataflow_finance_testing table.

```
SELECT
  FORMAT_TIMESTAMP("%Y %B %C  %R",CURRENT_TIMESTAMP()) AS date_time
  ,(SELECT count(*) FROM `course16_test.gwz_sales_pk`) AS sales_pk
  ,(SELECT count(*) FROM `course16_test.gwz_product_pk`) AS product_pk
  ,(SELECT count(*) FROM `course16_test.gwz_sales_margin_pk`) AS sales_margin_pk
  ,(SELECT count(*) FROM `course16_test.gwz_sales_margin_column-purchase_price`) AS
     sales_margin_column_purchase_price
  ,(SELECT count(*) FROM `course16_test.gwz_orders_pk`) AS orders_pk
 ,(SELECT count(*) FROM `course16_test.gwz_ship_pk`) AS ship_pk
  ,(SELECT count(*) FROM `course16_test.gwz_orders_operational_pk`) AS orders_operational_pk
  ,(SELECT count(*) FROM `course16_test.gwz_orders_operational_column-ship`) AS orders_operational_column_ship
  ,(SELECT * FROM `course16_test.gwz_orders_operational_conservation-ship1`) = (SELECT * FROM `course16_test.gwz_orders_operational_conservation-ship2`) AS orders_operational_conservation_ship
```

3) Create a scheduled query named dataflow_finance_testing_update to run dataflow_finance_testing_view every morning at 7:30 a.m. This will update the dataflow_finance_testing table with the latest test results.

Trigger an immediate manual run of your scheduled query by clicking on [Scheduled backfill](https://cloud.google.com/bigquery/docs/scheduling-queries#set_up_a_manual_run_on_historical_dates).



4) Analyse the results in dataflow_finance_testing. If there are errors, investigate the cause, correct them, and re-trigger an immediate manual execution of your scheduled query to ensure the corrections worked.


# **To Conclude:**

In this exercise, we learned how to manage a financial data pipeline by understanding data lineage and automating key processes. We started by mapping data lineage with a Directed Acyclic Graph (DAG) to see how everything connects, making troubleshooting and improvements easier.

Then, we set up data orchestration with scheduled queries to keep financial reports up-to-date automatically. We also automated tests to ensure data accuracy and reliability.

By the end, you should have a solid grasp of data lineage, orchestration, and the role of automated testing in maintaining a strong data pipeline.
