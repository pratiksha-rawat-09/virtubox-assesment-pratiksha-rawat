# calculative evidence for insights

## Insight 1

```sql
WITH monthly_sales AS (
    SELECT 
        EXTRACT(MONTH FROM Transaction_Date)::INT AS sales_month,
        SUM(Purchase_Amount) AS total_monthly_revenue
    FROM 
        ecommerce_transactions
    GROUP BY 
        1
),
avg_sales AS (
    SELECT 
        AVG(total_monthly_revenue) AS overall_avg_monthly_revenue
    FROM 
        monthly_sales
)
SELECT 
    ROUND(AVG(CASE WHEN sales_month IN (11, 12) THEN total_monthly_revenue END)::NUMERIC, 2) AS q4_avg_monthly_revenue,
    ROUND(m.overall_avg_monthly_revenue::NUMERIC, 2) AS overall_avg_monthly_revenue,
    ROUND(
        (
            (AVG(CASE WHEN sales_month IN (11, 12) THEN total_monthly_revenue END) - m.overall_avg_monthly_revenue) 
            / m.overall_avg_monthly_revenue * 100
        )::NUMERIC, 
    2) AS percentage_increase
FROM 
    monthly_sales, 
    avg_sales m
GROUP BY 
    m.overall_avg_monthly_revenue;
```
	
## Insight 2
```sql
	WITH age_groups AS (
    SELECT 
        CASE 
            WHEN Age BETWEEN 18 AND 24 THEN '18-24'
            WHEN Age BETWEEN 25 AND 34 THEN '25-34'
            WHEN Age BETWEEN 35 AND 44 THEN '35-44'
            WHEN Age BETWEEN 45 AND 54 THEN '45-54'
            WHEN Age >= 55 THEN '55+'
            ELSE 'Unknown'
        END AS age_group,
        Purchase_Amount
    FROM 
        ecommerce_transactions
),
demographic_summary AS (
    SELECT 
        age_group,
        COUNT(*) AS total_orders,
        SUM(Purchase_Amount) AS group_total_revenue,
        AVG(Purchase_Amount) AS avg_order_value
    FROM 
        age_groups
    GROUP BY 
        age_group
)
SELECT 
    age_group,
    total_orders,
    ROUND(group_total_revenue::NUMERIC, 2) AS total_revenue,
    ROUND(avg_order_value::NUMERIC, 2) AS avg_order_value,
    ROUND(
        (group_total_revenue / SUM(group_total_revenue) OVER () * 100)::NUMERIC, 
    2) AS percentage_of_total_revenue
FROM 
    demographic_summary
ORDER BY 
    total_revenue DESC;
```

## Insight 3
```sql	
	SELECT 
    EXTRACT(MONTH FROM Transaction_Date)::INT AS sales_month,
    TO_CHAR(Transaction_Date, 'Month') AS month_name,
    COUNT(Transaction_ID) AS total_orders,
    ROUND(SUM(Purchase_Amount)::NUMERIC, 2) AS total_monthly_revenue,
    ROUND(AVG(Purchase_Amount)::NUMERIC, 2) AS avg_order_value
FROM 
    ecommerce_transactions
GROUP BY 
    1, 2
ORDER BY 
    total_monthly_revenue DESC;
```

## Insight 4

```sql
	WITH category_summary AS (
    SELECT 
        Product_Category,
        COUNT(Transaction_ID) AS total_orders,
        SUM(Purchase_Amount) AS total_category_revenue,
        AVG(Purchase_Amount) AS avg_order_value
    FROM 
        ecommerce_transactions
    GROUP BY 
        Product_Category
)
SELECT 
    Product_Category,
    total_orders,
    ROUND(total_category_revenue::NUMERIC, 2) AS total_revenue,
    ROUND(avg_order_value::NUMERIC, 2) AS avg_order_value,
    -- Calculates percentage of total orders
    ROUND(
        (total_orders::NUMERIC / SUM(total_orders) OVER () * 100), 
    2) AS percentage_of_total_orders,
    -- Calculates percentage of total revenue
    ROUND(
        (total_category_revenue / SUM(total_category_revenue) OVER () * 100)::NUMERIC, 
    2) AS percentage_of_total_revenue
FROM 
    category_summary
ORDER BY 
    total_orders DESC;

```
	
## Insight 5

```sql
	WITH payment_summary AS (
    SELECT 
        Payment_Method,
        COUNT(Transaction_ID) AS total_transactions,
        SUM(Purchase_Amount) AS total_revenue,
        AVG(Purchase_Amount) AS avg_order_value
    FROM 
        ecommerce_transactions
    GROUP BY 
        Payment_Method
)
SELECT 
    Payment_Method,
    total_transactions,
    ROUND(total_revenue::NUMERIC, 2) AS total_revenue,
    ROUND(avg_order_value::NUMERIC, 2) AS avg_order_value,
    -- Percentage of total transactions across all payment methods
    ROUND(
        (total_transactions::NUMERIC / SUM(total_transactions) OVER () * 100), 
    2) AS percentage_of_transactions,
    -- Percentage of total revenue across all payment methods
    ROUND(
        (total_revenue / SUM(total_revenue) OVER () * 100)::NUMERIC, 
    2) AS percentage_of_revenue
FROM payment_summary
ORDER BY 
    avg_order_value DESC;
   ```
