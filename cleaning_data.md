# What issues will you address by cleaning the data? #

## *data has to be readable, concisely computable when applicable, and trimmed.*

- Certain columns need to be properly formatted EX: *dates should be TYPE DATE*.
- Dollar values need to be formatted to two decimals.
- Unnecessary/null columns should be dropped to avoid clutter.
- Missing values should be filled in when appropriate. Especially with categorical data that can be determined by observing other data.
- Categorical columns should be checked to make sure they weren't manually entered by a user. If they were then the values need to be made consistent. EX 'Canada' being 'CA' or vice versa.



# Queries: #
***Below, provide the SQL queries you used to clean your data.***

## Formatting prices to two decimal places.

If column was not created as a float then alter it:
```SQL
ALTER TABLE all_sessions
ALTER COLUMN product_price TYPE FLOAT;
```

Then divide by 1,000,000 and update

 ```SQL
UPDATE all_sessions SET product_price = product_price / 1000000;
 ```
 If needed round to two decimal places
 ```SQL
UPDATE all_sessions
SET product_price = ROUND(product_price::NUMERIC, 2)
WHERE product_price IS NOT NULL;
 ```

## Formatting dates to type date instead of int
unlike the top query going from a numerical data type to DATE or DATE_TIME requires an intermediate cast to a string data type before being cast to DATE.
```SQL
ALTER TABLE all_sessions
ALTER COLUMN date TYPE DATE USING DATE(date::TEXT);
```

## Clear clutter by dropping unwanted columns
*note that columns can also be dropped through pgAdmin's ui or in Excel if you inspect CSVs in Excel before importing*
```SQL
DROP COLUMN unwanted_column FROM table_you_are_cleaning;
```
**DROP ALL NULL COLUMNS**

## Fill Values
for all_sessions.currency_code the only imported values are null or 'USD' and we want all to be 'USD'
```SQL
UPDATE TABLE all_sessions SET currency_code = 'USD'
where currency_code = null;
```
*note that I verified all values were 'USD' by running*
```SQL
SELECT * from all_sessions WHERE currency_code = NULL;
--Copy a product_sku then
SELECT * FROM all_sessions WHERE product_sku = '<product_sku>';
``` 
*This will give you a small table showing that the price is the same where the currency_code for the corresponding sku is 'USD' or NULL. Rarely some values have different prices but I chose to keep them in as they have higher page_view and time_on_site stats leading me to think the reduced price was due to a sale/promotion. There are more patterns supporting this*
## Verify Consistency of Categorical values
Select a count of all country names, the order should be ASC on Count as variations/mispellings should have a lower occurence.
```SQL
SELECT COUNT(country) AS co, COUNTRY FROM all_sessions GROUP BY COUNTRY ORDER BY co, COUNTRY;
```
Skimming this over we can see that all countries are uniform.

However values like 'C??te dƒ??Ivoire' caught my eye.

That looks like the country 'Côte d'Ivoire' so we can assume the import had a hard time with the accented letters.

```SQL
SELECT country FROM all_sessions WHERE country LIKE '%?%' GROUP BY country;
```
Give us C??te dƒ??Ivoire and "R??union"

```SQL 
UPDATE all_sessions
SET country =
(
    CASE WHEN 
    country = 'C??te dƒ??Ivoire' then 'Cote dIvoire'
    ELSE 'Reunion'
    END
)
where country LIKE '%?%';
```
Do a quick check to see if we can fill in missing country values based on present city values.
```SQL
SELECT country, city FROM all_sessions WHERE country IS NULL;
```
In this case there are none.
## Remove White Space From product_sku
some product_sku values have white spaces. This removes them.
```SQL
UPDATE all_sessions
SET product_sku = REPLACE(product_sku, ' ', '');
```
## Taking full_visitor_id out of scientific notation
Similiar to the other TYPE conversions it needs multiple casts.
```SQL
ALTER TABLE all_sessions
ALTER column full_visitor_id TYPE BIGINT
USING CAST(CAST(full_visitor_id  AS FLOAT)/1000000000000 AS BIGINT)
```