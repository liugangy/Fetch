# Fetch
Language used: MySQL

Diagram:

![data model](https://github.com/liugangy/Fetch/blob/main/ERD.jpeg) 

Business Questions:
1. Which brand has the most spend among users who were created within the past 6 months?
2. Which brand has the most transactions among users who were created within the past 6 months?

```sql
With Users_past_6mon as (
Select _id
From users
Where createdDate between date_sub(current_date(), interval 6 month) and current_date()
),

receipts_filtered as(
Select _id as receipt_id
From receipts r
Join Users_past_6mon u on r.userId=u._id),

Items_with_brand_filtered as(
Select i.*, b.name as brand_name
From items i
Left join brand b on i.brandcode=b.brandcode
Join receipts_filtered r on r.receipt_id =i.receipt_id),

Items_brand_agg as(
Select 
brand_name, 
count(receipt_id) as num_transaction_per_brand, 
sum(finalprice) as total_spend_per_brand
From Items_with_brand_filtered
Group by brand_name),

Top_brand_most_transactions as(
Select brand_name as Top_brand_most_transactions
From
(Select brand_name, 
Rank() over (order by num_transaction_per_brand desc) as rk_tran
From Items_brand_agg) t1
Where rk_tran = 1
),

Top_brand_most_spend as(
Select brand_name as Top_brand_most_spend
From
(Select brand_name, 
Rank() over (order by total_spend_per_brand desc) as rk_spend
From Items_brand_agg) t1
Where rk_spend = 1

Select (Select Top_brand_most_transactions from Top_brand_most_transactions) as Top_brand_most_transactions,
(Select Top_brand_most_spend from Top_brand_most_spend) as Top_brand_most_spend;
```
-- Data Quality Check 1: Evaluating completeness of values in critical columns (Taking the Items table as an example)
```sql
select count(brandCode)*1.00/count(*) as non_null_percentage_brandCode
from items
```
The result shows that the brandcode column from the items table is missing 62% of the data.

-- Data Quality Check 2: Finding orphaned receipt ids that only exist in receipts table but not in items table
```sql
select r._id as receipt_id
from receipts r
left join items i on r._id = i.receipt_id
where i.receipt_id IS NULL
```

-- Email:

Hi (Name),

I hope this email finds you well. During a routine data quality checks of our receipt and item data, I identified two significant data quality issues that I wanted to bring to your attention. 
These issues could have implications for our reporting accuracy and overall data integrity.

1. Data Completeness Issue: The brandcode column from the items table is missing in 62% of the records. This is critical as the brandcode helps us link items to their respective brands.
2. Orphaned Receipt IDs: Some receipt IDs exist in the receipts table but not in the items table. This means we have receipts recorded without any associated items.
                         Any analysis or reports that rely on item-level data will be incomplete.

To effectively resolve these issues, I need to understand:

1. Are there known reasons why brandcode might be missing in such a large proportion of records?
2. Are there any processes or system integrations that could be leading to receipts being recorded without their associated items?
3. Are there any recent changes in data entry procedures or system updates that could be contributing to these issues?

As we address these data quality issues and look to enhance our data systems, I anticipate potential performance and scaling concerns:

1. Data Volume: Increasing data completeness may lead to higher data volume, impacting storage and retrieval times.
2. Processing Load: Ensuring all receipt-item relationships are accurately recorded will require additional processing power, especially during peak times.

To address these, I plan to:

1. Optimize our database queries and indexing strategies to handle larger volumes of data efficiently.
2. Implement more robust data validation checks at the point of entry to minimize the occurrence of these issues in the future.
3. Collaborate with our IT team to ensure our infrastructure can scale accordingly to handle increased data processing needs.

Your guidance and support in addressing these questions and accessing the necessary information will be crucial in resolving these data quality issues effectively.

Thank you for your attention to this matter. I look forward to your insights.

Best,
Rebecca
