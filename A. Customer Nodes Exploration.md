## :technologist::woman_technologist: Case Study #4: Data Bank - Customer Nodes Exploration

## Case Study Questions

1. How many unique nodes are there on the Data Bank system?
2. What is the number of nodes per region?
3. How many customers are allocated to each region?
4. How many days on average are customers reallocated to a different node?
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

***

###  1. How many unique nodes are there on the Data Bank system?

```sql
SELECT count(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
``` 
	
#### Result set:
![image](https://user-images.githubusercontent.com/77529445/165895245-c6b15626-c023-4d1a-9aaa-43cf8d3f1878.png)

***

###  2. What is the number of nodes per region?

```sql
SELECT 
	b.region_name, 
	COUNT(a.node_id) AS num_nodes
FROM customer_nodes a
LEFT JOIN regions b
ON a.region_id=b.region_id
GROUP BY b.region_name;
``` 
	
#### Result set:
![image](https://user-images.githubusercontent.com/125182638/225564201-8566c762-9ff8-4958-b186-71d9d7884814.png)

***

###  3. How many customers are allocated to each region?

```sql
SELECT
	a.region_id,
	b.region_name,
	COUNT(DISTINCT a.customer_id) AS total_cus
FROM customer_nodes a
LEFT JOIN regions b
ON a.region_id=b.region_id
GROUP BY a.region_id,b.region_name
ORDER BY a.region_id;
``` 
	
#### Result set:
![image](https://user-images.githubusercontent.com/125182638/225564471-558c12ed-4443-4fc2-a881-95c09e28418f.png)

***

###  4. How many days on average are customers reallocated to a different node?

```sql
SELECT 
	AVG(DATEDIFF(DAY, start_date, end_date)) AS avg_days
FROM customer_nodes
WHERE end_date!='9999-12-31';
``` 
	
#### Result set:
![image](https://user-images.githubusercontent.com/125182638/225564676-d893f552-f72a-4c6e-a254-b1508533a7bf.png)

***

###  5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
[UPDATING]

***
