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


***

**5. Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers:**
[UPDATING]
***

Thank you for reading. Do give me a 🌟 if you like this repo! 🙆🏻‍♀️
