## 🏦 Solution - B. Customer Transactions

**1. What is the unique count and total amount for each transaction type?**

````sql
SELECT	txn_type,
		COUNT (*) UNI_COUNT,
		SUM(txn_amount) TOTAL_AMT
FROM customer_transactions
GROUP BY txn_type
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/227535732-3d627438-9cf0-4d57-9b23-3c09de40eb25.png)

***

**2. What is the average total historical deposit counts and amounts for all customers?**

- Firstly, find the count of transaction and average transaction amount for each customer.
- Then, find the average of both columns where the transaction type is deposit.

````sql
WITH deposit AS
	(SELECT customer_id,
			txn_type,
			COUNT(*) TXN_COUNT,
			AVG(txn_amount)	AVG_AMT
	FROM customer_transactions
	GROUP BY customer_id, txn_type)
SELECT
		AVG(TXN_COUNT) AVG_HIS_COUNT,
		AVG(AVG_AMT)   AVG_BY_DEPOSIT
FROM deposit
WHERE txn_type ='deposit'
````
**Answer:**

![image](https://user-images.githubusercontent.com/125182638/227535874-bd67d2ee-5a76-43e6-a9b3-311bd6e12f53.png)

- The average historical deposit count is 5 and average historical deposit amounts are 508.61.

***

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

- First, create a `CTE` with output counting the number of deposits, purchases and withdrawals for each customer grouped by month.
- Then, filter the results to 
  - 2 or more deposits AND
    - 1 or more purchase(s) OR
    - 1 or more withdrawal(s) 
in a single month.

````sql
WITH txn_month AS
(
SELECT 
	   MONTH(txn_date) month_txn,
	   customer_id,
	   SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
	   SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
	   SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
FROM customer_transactions
GROUP BY 
	   MONTH(txn_date),
	   customer_id
)
SELECT 
	   month_txn,	
	   COUNT( DISTINCT customer_id) number_of_customers
FROM TXN_MONTH
WHERE (deposit_count >= 2 AND purchase_count = 1) or ( deposit_count >= 2 and withdrawal_count =1)
GROUP BY month_txn;
````

**Answer:**

![image](https://user-images.githubusercontent.com/125182638/227550507-a25f45f6-8fe2-4100-ad34-b32e347c54ff.png)
***

**4. What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.**

This is a particularly difficult question - with probably the most `CTE`s I have in a single query - there are 5 `CTE`s! 

I'm sure there's a shorter way to write the syntax, but I reckoned this is the best way as it allows me to build on my results. Take your time and run the table `CTE` by `CTE` to see the full picture and gain a complete understanding of the solution. 

````sql
-- CTE 1 - To identify transaction amount as an inflow (+) or outflow (-)
WITH monthly_balances AS (
  SELECT 
    customer_id, 
    (DATE_TRUNC('month', txn_date) + INTERVAL '1 MONTH - 1 DAY') AS closing_month, 
    txn_type, 
    txn_amount,
    SUM(CASE WHEN txn_type = 'withdrawal' OR txn_type = 'purchase' THEN (-txn_amount)
      ELSE txn_amount END) AS transaction_balance
  FROM data_bank.customer_transactions
  GROUP BY customer_id, txn_date, txn_type, txn_amount
),

-- CTE 2 - To generate txn_date as a series of last day of month for each customer
last_day AS (
  SELECT
    DISTINCT customer_id,
    ('2020-01-31'::DATE + GENERATE_SERIES(0,3) * INTERVAL '1 MONTH') AS ending_month
  FROM data_bank.customer_transactions
),

-- CTE 3 - Create closing balance for each month using Window function SUM() to add changes during the month
solution_t1 AS (
  SELECT 
    ld.customer_id, 
    ld.ending_month,
    COALESCE(mb.transaction_balance, 0) AS monthly_change,
    SUM(mb.transaction_balance) OVER 
      (PARTITION BY ld.customer_id ORDER BY ld.ending_month
      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS closing_balance
  FROM last_day ld
  LEFT JOIN monthly_balances mb
    ON ld.ending_month = mb.closing_month
      AND ld.customer_id = mb.customer_id
),

-- CTE 4 - Use Window function ROW_NUMBER() to rank transactions within each month
solution_t2 AS (
  SELECT 
    customer_id, ending_month, 
    monthly_change, closing_balance,
    ROW_NUMBER() OVER 
      (PARTITION BY customer_id, ending_month ORDER BY ending_month) AS record_no
  FROM solution_t1
),

-- CTE 5 - Use Window function LEAD() to query value in next row and retrieve NULL for last row
solution_t3 AS (
  SELECT 
    customer_id, ending_month, 
    monthly_change, closing_balance, 
    record_no,
    LEAD(record_no) OVER 
      (PARTITION BY customer_id, ending_month ORDER BY ending_month) AS lead_no
  FROM solution_t2
)

SELECT 
  customer_id, ending_month, 
  monthly_change, closing_balance,
  CASE WHEN lead_no IS NULL THEN record_no END AS criteria
FROM solution_t3
WHERE lead_no IS NULL;
````

**Answer:**

<img width="634" alt="image" src="https://user-images.githubusercontent.com/81607668/130431426-1882daec-8c93-4818-b041-943883aa21cb.png">

***

**5. Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers:**
[UPDATING]
***

Thank you for reading. Do give me a 🌟 if you like this repo! 🙆🏻‍♀️
