## My Question 1: How many buying customers came from which channel groupings?

SQL Queries:
```SQL
SELECT COUNT(channel_grouping),
			channel_grouping
FROM analytics
WHERE units_sold IS NOT NULL
GROUP BY channel_grouping
ORDER BY COUNT(channel_grouping) DESC
```

Answer:

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064360028712214528/image.png)

## My Question 2: What is the total amount of units bought by the customers in these channel groupings?

SQL Queries:
```SQL
SELECT * FROM --Join to the last Query so we can compare the ratio of buying customers to units_sold
(
    SELECT COUNT(channel_grouping),
           channel_grouping
    FROM analytics
    WHERE units_sold IS NOT NULL
    GROUP BY channel_grouping
    ORDER BY COUNT(channel_grouping) DESC
) AS a
JOIN --This new query is nearly identical but grabs the sum of units_sold for each grouping.
(
    SELECT SUM(units_sold)
           total_sold,
           channel_grouping
    FROM analytics
    GROUP BY channel_grouping
    ORDER BY SUM(units_sold) DESC
) AS b
ON a.channel_grouping = b.channel_grouping
ORDER BY total_sold DESC;
```

Answer:

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064361501512712302/image.png)

## My Question 3: Can you identify products that should be restocked if possible?

SQL Queries:

```SQL
SELECT COUNT(product_name) count, --Select how often the product is viewed and the name of the product.
	   product_name
FROM all_sessions s
JOIN products p
on p.sku = s.product_sku
WHERE stock_level = 0 --ONLY WHERE IT'S NOT IN STOCK
GROUP BY product_name
HAVING COUNT(product_name) > 5 -- Enter in a cutoff that illustrates a level of demand worth acting on (1 person viewing an out of stock item isn't profitable to restock).
ORDER BY count DESC;
```

Answer:

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064395278012129330/image.png)

This is a list of how many times a product that is out of stock is viewed by a visitor. 

After seeing this it's a no brainer to restock the wool cap!

## My Question 4: What conclusions can be drawn from the relationship the results from questions 1 and 2 have?

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064361501512712302/image.png)

Answer:

Referral customers buy 3.4 products each on average.

Organic search = 4.8

Direct = 6.2

Display = **23.3**

Paid Search = 2.9

Affiliates = 6.7

Social = 2.7

Customers acting on display sales are extremely valuable. Despite having the lowest count of buying customers per grouping they buy the 4th most products. Anything that can be done to generate more traffic from this grouping should be.

Referral's consistently buy and have a respectable average of products bought per customer. of all groupings, gaining more referrals will be the most beneficial. Even if you're paying for them you will have the most consistent ROE.

Despite being 54% of the referral groups size, the organic search group buys 77% of the amount of products the referral group does. This is a great sign of a healthy natural customer base that could grow further from general advertising / efforts to attract new users.

# I misread the assignment file and answered the example questions

## Question 1: find all duplicate records

SQL Queries:
```SQL
SELECT COUNT(*) AS count,
	   full_visitor_id
FROM all_sessions
GROUP BY full_visitor_id
HAVING COUNT(full_visitor_id) > 1
ORDER BY count DESC;
```

Answer: 

This will output 1,644 full_visitor_ids in all_sessions ordered by the most frequent duplication. This query will work on any table with full_visitor_id and of course full_visitor_id can be changed to something else like 'sku' to work with the product table.

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064333842267254884/image.png)

```SQL
SELECT SUM(count) from
(
	SELECT COUNT(*) AS count,
		   full_visitor_id
	FROM all_sessions
	GROUP BY full_visitor_id
	HAVING COUNT(full_visitor_id) > 1
	ORDER BY count DESC
) as subq;
```
This will output 14,684 as the total number of duplicate records.

If we subtract 1,644 from 14,684 we get 13,042 the number of duplicate records ONLY REMOVING the duplications, leaving the original.

## Question 2: find the total number of unique visitors (`fullVisitorID`)

SQL Queries:
```SQL
SELECT COUNT(DISTINCT full_visitor_id)
from all_sessions;
```
Answer:
There are 2094 unique full_visitor_ids

2,094 + 13,042 = 15,136. The same output we get from the query:
```SQL
 select count(full_visitor_id) from all_sessions
 ```

## Question 3: find the total number of unique visitors by referring sites

SQL Queries:
```SQL
select count(full_visitor_id)
from analytics
where channel_grouping = 'Referral'
and
full_visitor_id in
(
	SELECT DISTINCT full_visitor_id from all_sessions
)
```

Answer:
217 unique visitors.


## Question 4: find each unique product viewed by each visitor

SQL Queries:
```SQL
SELECT visit_id,
	   product_name
from all_sessions
group by visit_id,product_name
order by visit_id;
```

Answer:

![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064353394250235934/image.png)

this returns an ordered list of every visit_id and what they looked at in that session.

## Question 5: compute the percentage of visitors to the site that actually makes a purchase

SQL Queries:
```SQL
SELECT DISTINCT visit_id FROM analytics --37887
SELECT DISTINCT visit_id FROM analytics where units_sold is not null --4891
```

Answer:

12.9% of people buy a product when they visit the store.