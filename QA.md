# What are your risk areas? Identify and describe them.

## I'd say the main risk is the columns relating to orders / revenues.

- For example look at this table:
- ![alt text](https://cdn.discordapp.com/attachments/1063653051602321462/1064371704161382460/image.png)
- It seems product_revenue = quantity * price + a hidden amount (I am basing this on how 3.5 * 50 = 175 and revenue for that row is 176.4)
- What is total_transaction_revenue then? I first assumed it was the total revenue that the visid_id / full_id generated however adding both as a filter this is still the only row that shows up.
- There's tons of information regarding revenue that seems to have no connection to the other data and I'm not too confident what light I can hold it in. 
- Doing analytics.units_sold * analytics.unit_price provides way different results than sum(total_transaction_revenue) and so on.

## Another large risk is the amount of mismatched visit_ids and full_visitor_ids between analytics and all_sessions
- My initial understanding of the relationship analytics and all_sessions had is all_sessions is the customer visiting the domain and analytics is a breakdown of every page that customer views in the domain and how they interact with it.
- As I continued working with it I became less confident in that relationship. If that's so what is the reason for all the holes in matching IDs? Why do rows in all_sessions have a product name and details attatched to it. In analytics it's not uncommon to have one visit_id / full_visitor_id be listed many different times with all the same data except for the price. Joining with all_sessions implies they're looking at the exact same item but wildly different prices on it. Is this a bidding website maybe?

## strange instances of prices by SKU
- using a query like 
```SQL
select count(product_price), product_price, product_sku from all_sessions group by product_sku, product_price order by product_sku desc
```
- we get an output where we can find groupings of different prices per SKU.

- ![ALT TEXT](https://cdn.discordapp.com/attachments/1063653051602321462/1064383259183685682/image.png)

- ordering this specific SKU by date we can see it is pretty normal. It's original price was $19.99 and we can assume that the lower prices are due to it going on sale every now and then. However you can find some SKUs that have very strange pricing tied to them. Whether it be really heavy discounts or some sales at nearly double the regular price.

**The other risk areas were pretty easy to handle these ones I mentioned I feel additional context about the data whether it comes from the fictional client or your fictional teamate/superior is nescessary.** 

sku formatting came off as a risk area at first but I then realized there's a reason for 'GG%' and non 'GG%' SKUs

I thought the date format as INT would be an issue so I changed them to DATE anticipating questions regarding date comparisons but there were none.

Data is very skewed and limited for many countries/cities. For example you'd look up a country/city and find they've only bought 2 products, 50 of one and 2 of the other. 

Data is very skewed in favour of the U.S. Many U.S. Businesses have a large percentage of their customers residing in the U.S. as well so this is not out of the ordinary by any means but will have to be taken into consideration for certain questions that could be asked.

# QA Process:
Describe your QA process and include the SQL queries used to execute it.

## Example starting_with_data.md - My Question 3: Can you identify products that should be restocked if possible?

- In this query I filter results based on if the stock level = 0
```SQL
SELECT * FROM products WHERE stock_level = 0;
-- This returns 190 rows. We can also see a pattern in SKUs. The majority don't have 'GG' at the start. This could mean that SKUs stripped of GG have been designated as something the vendor is either unable to get anymore, it's not profitable to sell anymore, or they're choosing to not sell it for some other reason. If there's enough demand maybe it could be brought back however.
SELECT * FROM products WHERE sku NOT LIKE 'GG%';
-- This query returns 170 rows all with stock_level = 0 reinforcing the above comment.

SELECT COUNT(product_name) count, -- This is the original query with one change.
	   product_name,
	   product_sku
FROM all_sessions s
JOIN products p
ON p.sku = s.product_sku
WHERE stock_level = 0
OR
product_price = 0
GROUP BY product_name, product_sku
-- There was a having clause here but we comment it out to show all rows this returns.
ORDER BY count DESC;

-- This returns only 103 rows, with a higher looking percentage of GG SKUs.

SELECT * FROM products WHERE sku = 'GGOEGHPA003010';
-- Running these on some GG SKUs we see they are still null however.

SELECT * FROM all_Sessions WHERE product_sku = '9180850';
-- Here we take a sku from the earlier products query and see if it exists in all_sessions. It does not! This implies that this has been out of stock/discontinued/etc for a while and either doesn't even have a page anymore or no one has looked at it.
```
All of these tests support the result of the original query.

In summary I try and do the 'count check test' where I take a count of the column with what I think the data is that's taken from the join query and a count of the data returned by the join query. I find a difference so I investigate it and find it justified due to verifying a lack of occurences in all_sessions. This suggests the products table outdates the all_sessions table and is not cleaned when products stop being sold/discontinued.