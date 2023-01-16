Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

For country only:
```SQL
SELECT SUM(units_sold) AS
	level_of_transaction,
	country
FROM analytics a
JOIN all_sessions s
ON a.visit_id = s.visit_id
GROUP BY country
HAVING SUM(units_sold) IS NOT NULL
ORDER BY level_of_transaction DESC
```
For city and country:
```SQL
SELECT SUM(units_sold) AS
	level_of_transaction,
	country,
	city
FROM analytics a
JOIN all_sessions s
ON a.visit_id = s.visit_id
GROUP BY country, city
HAVING SUM(units_sold) IS NOT NULL
ORDER BY level_of_transaction DESC
```

Answer:

First Query:

![Alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1063671221541142538/image.png)

Second Query:

![Alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1063670838311796827/image.png)

**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
For Countries Only:

```SQL
SELECT ROUND(AVG(UNITS_SOLD),2) as 
	   average,
	   s.country
FROM analytics a
JOIN all_sessions s
ON a.visit_id = s.visit_id
GROUP BY country
HAVING AVG(UNITS_SOLD) IS NOT NULL
ORDER BY average DESC;
```

For Countries and Cities:

```SQL
SELECT ROUND(AVG(UNITS_SOLD),2) as 
	   average,
	   s.country,
	   s.city
FROM analytics a
JOIN all_sessions s
ON a.visit_id = s.visit_id
GROUP BY country,
	  city
HAVING AVG(UNITS_SOLD) IS NOT NULL
ORDER BY average DESC;
```


Answer:

First Query:

![ALT text](https://cdn.discordapp.com/attachments/1063653051602321462/1063673049938935839/image.png)

Second Query:

![ALT text](https://cdn.discordapp.com/attachments/1063653051602321462/1063672746103545886/image.png)

***(33 total lines)***

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

I mainly used:
```SQL
SELECT COUNT(product_category) as count,
	   product_category,
	   city,
	   country
FROM analytics a
JOIN all_sessions s
ON a.visit_id = s.visit_id
--where country = 'United States'
GROUP BY product_category,
		 city,
		 country
HAVING AVG(UNITS_SOLD) IS NOT NULL
ORDER BY country, city, count DESC;
```
You can uncomment the WHERE clause to do country specific stats, add and remove city, or country as well and play around with it then do simple math on the results to get percentages.

Answer:

Yes! 

For example the US makes up 37.8% of Home/Shop By Brand/Youtube purchases,

50.6% of Home/Apparel/Men's/Men's-T-Shirts/ purchases,

and 74.2% of Home/Electronics/ purchases.

The majority of things New York buys is Home/Shop by Brand/,

San Francisco's is Home/Accessories/Fun/,

Office supplies bought in Berlin make up 80.9% of everything Germany buys.

***The list could go on forever.***



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

*With this relatively small database you can get away with not using a CTE and the results will still be digestible (my original query is the query inside top_table without ROW_NUMBER( )). However this is not scalable and returning this selection for every single product_name people from that city have bought will quickly become too overwhelming to digest. Using this CTE only returns the top selling.*
```SQL
--TO FIND TOP SELLING IN CITY
WITH top_table AS
(
	SELECT count(product_name) count,
				city,
				country,
				product_name,
			    ROW_NUMBER() OVER (PARTITION BY city ORDER BY count(product_name) DESC) AS row_num
	FROM analytics a
	JOIN all_sessions s
	ON a.visit_id = s.visit_id
	WHERE city != 'not available in demo dataset' --If we didn't have this N/A would occur frequently.
	GROUP BY product_name, city, country
	HAVING AVG(UNITS_SOLD) IS NOT NULL
	ORDER BY count DESC
)
select * from top_table where row_num = 1 order by country
```

```SQL
--TO FIND TOP SELLING IN COUNTRY
WITH top_table AS
(
	SELECT count(product_name) count,
				country, --REMOVE CITY AND PARTITION BY COUNTRY INSTEAD OF CITY
				product_name,
			    ROW_NUMBER() OVER (PARTITION BY country ORDER BY count(product_name) DESC) AS row_num
	FROM analytics a
	JOIN all_sessions s
	ON a.visit_id = s.visit_id
	WHERE city != 'not available in demo dataset' 
	--^Since city is now irrelevant we can also choose to remove this filter but it will have a very large influence
	GROUP BY product_name, country --DO NOT GROUP BY CITY
	HAVING AVG(UNITS_SOLD) IS NOT NULL
	ORDER BY count DESC
)
SELECT * FROM top_table WHERE row_num = 1 ORDER BY country, count DESC
```

Answer:

CITY/COUNTRY QUERY:

![ALT text](https://cdn.discordapp.com/attachments/1063653051602321462/1064009299048808509/image.png)

COUNTRY QUERY:

![ALT text](https://cdn.discordapp.com/attachments/1063653051602321462/1064008849545240606/image.png)

**PATTERNS:**

(We can also pair this as a further insight into question three.)

Google merch is extremely popular accross the world.

Mountain View and Chicago seem to have similiar buying habits. Maybe advertizers could consider the two in a similiar light?

Look at San Fran. In Question 3 I stated their most purchased category was 'Fun.' They've only been buying one 'Fun' product and that's the 'Windup Android.' If you advertized this product in the San Fran area you could respect a high response of 'Oh, that's that fun Windup Android I've seen around.' One instance isn't really a pattern of course however with larger data it would be many more 'Fun' products they've been buying effectively providing a pattern of 'fun' products they like which can be used to infer which products they might also like.

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```SQL
--To judge impact we need to determine a total
WITH rev_table AS
(
		SELECT ROUND(SUM(units_sold * product_price)::NUMERIC,2) AS sum1,
	city,
				country
	FROM analytics a
	JOIN all_sessions s
	ON a.visit_id = s.visit_id
	WHERE city != 'not available in demo dataset' 
	GROUP BY country,
			 units_sold,
			 city
	HAVING UNITS_SOLD IS NOT NULL
	ORDER BY sum1 DESC
)
SELECT SUM(sum1)
FROM rev_table
WHERE sum1 IS NOT NULL;
-- This outputs 5302.96
```
Now to get the percentages. The percentages in the column Impact will represent how each city's revenue in the U.S. contritubes to the entire revenue in the U.S.
```SQL
WITH rev_table AS
(
		SELECT ROUND(SUM(units_sold * product_price)::NUMERIC,2) AS sum1,
	city,
				country
	FROM analytics a
	JOIN all_sessions s
	ON a.visit_id = s.visit_id
	WHERE city != 'not available in demo dataset' 
	GROUP BY country,
			 units_sold,
			 city
	HAVING UNITS_SOLD IS NOT NULL
	ORDER BY sum1 DESC
)
SELECT CONCAT(ROUND(((sum1 / 5302.96)*100),2),'%') AS impact,
		city,
	   country
FROM rev_table
WHERE sum1 IS NOT NULL;
```


Answer:

The last Query will output:

![ALT text](https://cdn.discordapp.com/attachments/1063653051602321462/1064326562650001470/image.png)






