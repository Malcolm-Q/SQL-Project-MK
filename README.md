# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
**My goals are to have a clean, properly formatted, easy to work with database**
### - Database should be clean.
### - Properly formatted data types.
### - Writing computations should be easy and data should be easily digestible. (No excessive casting for example)
### - .md files should make use of all markdown has to offer to be clear and organized.

## Process
### - Download and open CSVs, spend some time scrolling through them and trying to digest the columns and how they relate to eachother.
### - Create tables, designate primary keys to pre-existing columns on tables where appropriate. (Product and sales tables have unique SKUs).
### - Once imported set up auto incrementing primary keys for analytics and all_sessions.
### - Perform the data cleaning process as documented.
### - Generate ERD to visualize database and how to set foreign keys.
### - Set foreign keys. visit_id connects analytics and all_sessions, sku connects all_sessions to the rest of the tables.
### - begin answering questions in .md file

## Results
### My experience with this data I found that I could provide really great 'anonymous information' (information from analytics not joined with all_sessions mostly) like 'How many products does the average user buy when they visit the website?' Joining it with all_seasons to get personal information like city and country however I would lose a lot of those rows due to a lack of visit_id and full_visitor_id matches. I could get more personal information out of just all_seasons but it's not nearly the same amount of data.

## Challenges 
***I expand on these issues in QA.md***
### - Not being able to communicate with the fictional client that provided this data, or a teammate / superior who has worked with them like in a real life situation led to me creating my own context in some situations and being unable to decifer certain columns.
### - visit_id/full_visitor_id mismatches/holes when I perceived analytics and all_sessions relationship to not have any holes. I thought all_sessions was the session as a whole then analytics was all activity in that session. If that was the case though, analytics couldn't have visit_ids and full_visitor_ids that don't match up with all_sessions'.
### - poorly named/hard to understand dollar amount columns (total_transaction_revenue mainly)

## Future Goals
### It's 8:47PM on Sunday at the time of writing this but if I could start over I think I would spend more time cleaning / feeling out the data. Near the end once I started getting more concerns about the challenges above the quote 'data science is 80% cleaning 20% analysis and reporting' kept popping into my mind. I figure it must have been something I overlooked / made a mistake on that led to problems between analytics and all_sessions.
