---
title: "Creating a Data Pipeline Using Snowflake and DBT"
excerpt: "In this project we use the TCP-H Dataset provided by Snowflake to create a data pipeline"
#"<img src='/images/Accuracy-and-Loss-2.png'>"
collection: portfolio
---

I began this project by setting up the foundational elements in Snowflake—a warehouse, database, and schema—to ensure my data pipeline would have a robust and organized structure. I chose Snowflake for its ability to handle large datasets efficiently and for its straightforward integration with dbt (data build tool). Once my Snowflake environment was ready, I installed and configured dbt locally because it offers a clear framework for version-controlling SQL transformations and enforcing modeling best practices.

My goal was to build a pipeline around the TPC-H dataset supplied by Snowflake, focusing on three key tables: orders, lineitems, and customers. I staged the incoming raw data into “stg” (staging) views—named stg_tpch_orders, stg_tpch_lineitems, and stg_tpch_customer—so I could clean and transform the data in one place. Staging views serve as a foundation: they let me isolate any data cleaning or minor transformations before moving onto more advanced modeling. This layer also keeps my final models simpler and easier to maintain.

## From these staging views, I built several models:
int_order_items and int_order_items_summary (fact tables focused on the details of orders and their aggregated information)
dim_customer_orders (a dimension table capturing customer-related attributes)
fct_orders (my main fact table for orders)
I decided to structure these models in a layered approach:

Staging Layer (stg_*) – Minimal transformations and cleaning.
Intermediate/Facts/Dimensions (int_, fct_, dim_*) – Rich transformations, aggregations, and final business logic.
A key design choice was to create a surrogate key for the line items. The TPC-H dataset inherently uses a composite key (orderkey + linenumber) to identify line items. However, I wanted to give each record a single-column primary key for easier referencing in analytics tools. To do this, I leveraged the dbt_utils package and combined orderkey and linenumber into a surrogate primary key within the int_order_items table.

## Testing Strategy
I set up the following dbt tests to ensure data integrity:

### Orders table:
not_null test to guarantee no missing values in critical columns.
unique test to confirm that each order is uniquely identified.

### Lineitems table:
relationships test to check that the foreign key l_orderkey in the line items table corresponds to a valid order in the orders table.

### Customers table:
I decided not to include any tests initially, to keep the setup simple and direct my attention toward the more critical relationships between orders and line items.
By enforcing these tests, I can catch potential data quality issues right away, which is especially important if this pipeline scales to more extensive datasets or more complex transformations.

### Macro for Discounted Amount
In my transformation logic, I also developed a macro that calculates a discounted amount, which I used inside the int_order_items model. Macros allow me to define reusable SQL snippets and keep my codebase DRY (Don’t Repeat Yourself). Calculating a discounted price once and then referencing it across multiple models made my transformations more maintainable and standardized.

## Lineage Graph

![Image](images/lineage-graph.png)

The lineage graph (shown above) highlights how data flows from the staging views to the intermediate tables and then on to the fact and dimension layers. This visualization helps me (and any collaborators) understand dependencies and the overall structure of the pipeline. It reveals how stg_tpch_orders, stg_tpch_lineitems, and stg_tpch_customer feed into the dimension and fact tables:

int_order_items → int_order_items_summary → fct_orders
dim_customer_orders → fct_orders

## Documentation
Finally, I documented all these models, tests, and macros within dbt, and published the dbt docs as a website through GitHub Pages. You can find a more detailed overview of each table, column definitions, and dependencies at my dbt docs site. This documentation not only improves transparency for others reviewing or collaborating on the project but also serves as a reference point for me when iterating on the pipeline in the future.

In summary, I built this data pipeline in Snowflake with dbt to clean, transform, and model the TPC-H dataset. Every decision, from creating staging layers to implementing tests, surrogate keys, and macros, was made to ensure a reliable, maintainable, and scalable solution. By combining Snowflake’s performance with dbt’s structured approach to SQL modeling, I have a robust pipeline that is easy to extend and troubleshoot.