~~~ SQL
/*1. Data type of columns in a table*/
/*Sellers*/

SELECT * FROM Target.INFORMATION_SCHEMA.COLUMNS
where table_name = "Sellers";

~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/f131ecba-0ff5-495b-87b0-18d5b4c754f5)


~~~ SQL
/*Code to count row numbers*/

SELECT COUNT(*) AS num_rows
FROM `Target.Sellers`
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/800becaf-d95c-43b5-acdc-58a35dc45423)


~~~ SQL
/*Time period for which the data is given*/

SELECT MIN (order_purchase_timestamp) AS min_value,
MAX(order_purchase_timestamp) AS max_value
FROM `Target.Orders
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/50ca9f80-020d-46f7-94d7-979d9302593d)



~~~ SQL
/*Cities and States of customers ordered during the given period*/
/*City*/
SELECT Distinct geolocation_city
FROM `Target.Geolocation`
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/b9af39da-d88f-43ed-bcc9-256bff1f8323)



~~~ SQL
/*State*/
SELECT Distinct geolocation_state
FROM `Target.Geolocation`
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/54349f40-5215-4d7f-9251-29f2d34b4d9e)

**2. In-depth Exploration**
~~~ SQL
/*2.1 Is there a growing trend on e-commerce in Brazil? How can we describe a complete scenario? Can we see some seasonality with peaks at specific months?*/
SELECT
COUNT(Distinct order_id) AS total_orders,
EXTRACT(Month from order_purchase_timestamp) As Month,
EXTRACT(Year from order_purchase_timestamp) As Year FROM`Target.Orders`
GROUP BY month, year
ORDER BY year, month asc ;
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/c3102d55-5707-4adc-89b8-226bc1541084)

Insight-
Yes, there is a growing trend in e-commerce in Brazil. From the end of 2016 to the end of 2018 (except the 9th and 1oth month of 2018) there was an increase in unique Order
IDs in Brazil.

Regarding seasonality, there are some peaks at speciﬁc months in Brazil. In February 2017 & November 2017 we can see peak in orders signiﬁcantly.

Recommendation- there was a speciﬁc event or promotion that caused an increase in demand in Feb & Nov 2017.. The business should investigate the cause of this spike and try to replicate it in the future.

~~~ SQL
/*2.2 What time do Brazilian customers tend to buy (Dawn, Morning, Afternoon or Night)?*/
SELECT Buying_time, count(Buying_time) as Count_of_buying
FROM
(SELECT
order_purchase_timestamp, CASE
  when Hour_field = 5 and mins_field between 0 and 59 then "Dawn"
  when Hour_field between 6 and 11 and mins_field between 0 and 59 then "Morning"
  when Hour_field between 12 and 17 and mins_field between 0 and 59 then "Afternoon"
  Else "Night"
END as Buying_time FROM
(SELECT
     order_purchase_timestamp,
     EXTRACT(HOUR FROM order_purchase_timestamp) AS HOUR_FIELD,
     EXTRACT(MINUTE FROM order_purchase_timestamp) AS MINS_FIELD,
     EXTRACT(SECOND FROM order_purchase_timestamp) AS SECS_FIELD FROM
     `Target.Orders`))
     Group By Buying_time

~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/37171c89-1184-493d-a7f7-7fdf109bfccc)

~~~ SQL
/*State*/

~~~










