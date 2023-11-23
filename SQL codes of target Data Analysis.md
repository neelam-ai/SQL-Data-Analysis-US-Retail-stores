**1. Exploratory analysis**
~~~ SQL
/*1.1 Data type of columns in a table*/
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
/*1.2Time period for which the data is given*/

SELECT MIN (order_purchase_timestamp) AS min_value,
MAX(order_purchase_timestamp) AS max_value
FROM `Target.Orders
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/50ca9f80-020d-46f7-94d7-979d9302593d)



~~~ SQL
/*1.3 Cities and States of customers ordered during the given period*/
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

**2.1 Is there a growing trend on e-commerce in Brazil? How can we describe a complete scenario? 

2.2 Can we see some seasonality with peaks at specific months?**
~~~ SQL
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


**2.3 What time do Brazilian customers tend to buy (Dawn, Morning, Afternoon or Night)?**
~~~ SQL
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
Nighttime purchases had the highest count of 38,652, followed by afternoon purchases with a count of 38,361. Morning and Dawn purchases had the lowest count with 22,240.

**3. Evolution of E-commerce Orders in Brazil Region**

**3.1 Get month-on-month orders by states**

~~~ SQL
SELECT
customer_state,
count(distinct(order_id)) as Customer_count, Month,
Year FROM (SELECT
o.customer_id,
t.customer_state,
o.order_id,
EXTRACT(Month from order_purchase_timestamp) As Month,
EXTRACT(Year from order_purchase_timestamp) As Year FROM `Target.Orders` AS o
LEFT JOIN `Target.Customers` AS t ON o.customer_id = t.customer_id) GROUP BY customer_state, Month, Year
ORDER BY customer_state, Year,Month
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/5a0f814d-1ed5-4582-a014-13c1c529bae0)


**How are the customers distributed across all the states?**

~~~ SQL
SELECT
customer_state,
count(distinct(customer_id)) as Customer_count FROM `Target.Customers`
GROUP BY customer_state ORDER BY Customer_count DESC
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/8ce36e64-ad2e-4243-8971-567d978f3861)

**Insight**
São Paulo (SP) has the highest number of customers with a count of 41,746, Rio de Janeiro (RJ) has the second highest with 12,852 customers and Minas Gerais (MG) is the third with 11,635 customers.

In Total 27 states, the top 12 states have more than 1000 customers. 13th to 24th have customers of more than 100 but less than 1000.
The bottom 3 states have less than 100 customers.
Recommendation-Focus on retaining customers in states with high customer counts by offering promotions, discounts, or loyalty programs to incentivize repeat purchases.

-For states with low customer counts, consider targeted marketing campaigns to
acquire new customers, such as social media advertising, inﬂuencer partnerships, or referral programs.

**4. Impact on Economy: Analyze the money movement by e-commerce by looking at order prices, freight and others**


**4.1 Get % increase in cost of orders from 2017 to 2018 (include months between Jan to Aug only)**
**You can use “payment_value” column in payments table**
~~~ SQL
With Base as (select
EXTRACT(Year from order_purchase_timestamp) As Year, Sum(payment_value)as revenue
From `Target.Orders` as t
inner join `Target.Payments`as p on t.order_id = p.order_id
where EXTRACT(Month from order_purchase_timestamp) between 1 and 8 GROUP BY Year),
base2 as (Select *, Lag(revenue) over (order by Year)as previous_revenue from base )
SELECT *, (revenue-previous_revenue)/previous_revenue*100 as per_INC from base2
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/af0ef09f-e8e7-463d-936e-13ff711ddf36)
**Insight**- The revenue in 2018 is signiﬁcantly higher than in 2017, with a year-over-year increase of 136.98%.
**Recommendation**- Use the revenue data to inform budgeting and resource allocation decisions, such as increasing investments in marketing or expanding product offerings in high-growth areas.



**4.2 and 4.3 Mean & Sum of price and freight value by a customer state**

~~~ SQL
SELECT
customer_state,
SUM(price) AS Price_Sum,
SUM(freight_value) AS Freight_Sum,
AVG(price) AS Avg_Price,
AVG(freight_value) AS Avg_Freight FROM
`Target.order_items` AS items INNER JOIN
`Target.Orders` AS o ON
items.order_id = o.order_id INNER JOIN
`Target.Customers` AS c ON
o.customer_id = c.customer_id GROUP BY
customer_state Order by price_sum
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/8e6545c4-5c62-4033-8570-5873dc8ff799)

Insight-
-The highest total price sum is from the state of São Paulo (SP), which is more than twice the price sum of the second-highest state, Rio de Janeiro (RJ).
-The highest average price is also from the state of São Paulo (SP), indicating that customers from this state tend to purchase higher-priced items.
-The lowest total price sum is from the state of Espírito Santo (ES).
-The highest total freight sum is from the state of Minas Gerais (MG), followed by São Paulo (SP) and Rio de Janeiro (RJ).
-The highest average freight is from the state of Bahia (BA).

Recommendation-
-The states with higher average prices and freight values may be good targets for marketing campaigns for higher-end products, while those with lower values may be better suited for more affordable products.

**5.Analysis of sales, freight, and delivery time**

**5.1 Calculate days between purchasing, delivering and estimated delivery**

~~~ SQL
SELECT
 	order_purchase_timestamp,
 	order_estimated_delivery_date,
 	order_delivered_customer_date,
 	estimated_day,
 	Delivery_time,
     (estimated_day-Delivery_time) AS buffer_day FROM (
     SELECT
 	order_purchase_timestamp,
 	order_estimated_delivery_date,
 	DATE_DIFF(order_estimated_delivery_date, order_purchase_timestamp, DAY) AS estimated_day,
 	order_delivered_customer_date,
 	DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY) AS Delivery_time

FROM
 	`Target.Orders`
     WHERE
 	order_delivered_customer_date IS NOT NULL)
 	order by buffer_day
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/894e1ce7-08f0-463f-9470-aa7653300674)
**Insight** There is a late delivery of max 189 days and there are several orders having this late delivery.
Recommendation- we can analyze the reason for the late delivery and give the correct estimated_delivery_date to the customer.
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/dec9d493-bc76-46a1-869f-7f3ac6505194)
**Insight2**- After sorting data by Delivery_time we can see there is an order which is delivered on the same day or in 1 day.
Recommendation- We can encourage these customers to give good reviews on our online platform or in google my business.


**Find time_to_delivery & diff_estimated_delivery. Formula for the same given below:**

~~~ SQL
SELECT
 	order_purchase_timestamp,
 	order_delivered_customer_date,
 	DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY) AS time_to_delivery
FROM `Target.Orders`
     WHERE
 	order_delivered_customer_date IS NOT NULL ORDER BY
     time_to_delivery
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/3799c69c-9f19-4f6d-a1a3-2908ffec091b)
**Insight**
some orders being delivered within a day while others take over 200 days.
**Recommendation**
Optimize logistics processes and consider partnering with reliable carriers to ensure timely deliveries.



~~~ SQL
SELECT
order_purchase_timestamp, order_estimated_delivery_date,
DATE_DIFF(order_estimated_delivery_date, order_purchase_timestamp, DAY) AS diff_estimated_delivery
FROM
`Target.Orders` WHERE
order_delivered_customer_date IS NOT NULL ORDER BY
diff_estimated_delivery desc
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/5e09df40-d34d-4a09-8ada-117a2ace7bb5)



~~~ SQL
/*Group data by state, take mean of freight_value, time_to_delivery, diff_estimated_delivery*/

SELECT
customer_state, AVG(freight_value) AS Avg_Freight,
AVG(time_to_delivery) AS Avg_time_to_delivery, AVG(diff_estimated_deliver) AS Avg_diff_estimated_deliver, FROM
(SELECT
     customer_state,
     DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY) AS time_to_delivery,
     DATE_DIFF(order_estimated_delivery_date, order_purchase_timestamp, DAY) AS diff_estimated_deliver,
     freight_value FROM
     `Target.order_items` AS items INNER JOIN
     `Target.Orders` AS o ON
     items.order_id = o.order_id INNER JOIN
     `Target.Customers` AS c ON o.customer_id = c.customer_id
WHERE order_delivered_customer_date IS NOT NULL)
  GROUP BY customer_state
  Order by Avg_time_to_delivery desc
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/ceb8ba04-d57a-44a4-9640-98883f421c45)


~~~ SQL
/*Top 5 states with highest/lowest average freight value - sort in desc/asc limit 5*/
SELECT customer_state,AVG(freight_value) AS Avg_Freight
FROM
(SELECT
 	customer_state,
 	DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY) AS time_to_delivery,
 	DATE_DIFF(order_estimated_delivery_date, order_purchase_timestamp, DAY) AS diff_estimated_deliver,
 	freight_value
  FROM
 	`Target.order_items` AS items
     INNER JOIN
 	`Target.Orders` AS o
     ON
 	items.order_id = o.order_id
     INNER JOIN
 	`Target.Customers` AS c
     ON
 	o.customer_id = c.customer_id
     WHERE
 	order_delivered_customer_date IS NOT NULL) GROUP BY customer_state
ORDER BY Avg_Freight DESC LIMIT 5;
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/fe860418-66ba-4661-b668-73fab3999fca)



~~~ SQL
/*Top 5 states with highest/lowest average time to delivery*/
SELECT
customer_state,
AVG(time_to_delivery) AS Avg_time_to_delivery FROM
(SELECT
customer_state,
DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY) AS time_to_delivery
FROM
`Target.order_items` AS items INNER JOIN
`Target.Orders` AS o ON
items.order_id = o.order_id INNER JOIN
`Target.Customers` AS c ON
o.customer_id = c.customer_id WHERE
order_delivered_customer_date IS NOT NULL) GROUP BY customer_state
ORDER BY Avg_time_to_delivery DESC LIMIT 5;
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/b966110d-2006-4d6e-aa6a-3c7b08bb997a)



~~~ SQL
/*Top 5 states where delivery is really fast/ not so fast compared to estimated date*/
SELECT
customer_state,
AVG(diff_estimated_delivery) as diff_estimated_delivery FROM
(SELECT
customer_state, order_estimated_delivery_date, order_delivered_customer_date,
DATE_DIFF(order_estimated_delivery_date,order_delivered_customer_date , DAY) AS diff_estimated_delivery
FROM
`Target.order_items` AS items INNER JOIN
`Target.Orders` AS o ON
items.order_id = o.order_id INNER JOIN
`Target.Customers` AS c

ON
o.customer_id = c.customer_id WHERE
order_delivered_customer_date IS NOT NULL) GROUP BY customer_state
ORDER BY diff_estimated_delivery ASC Limit 5
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/962d24d7-b35f-4883-967e-63074516f4af)


**6.Payment type analysis:**


**6.1 Month over Month count of orders for different payment types**
~~~ SQL
With Base as(SELECT Payment_type,
Month, Year,
count(distinct(order_id)) as Customer_count,
FROM (SELECT
o.customer_id, p.payment_type, o.order_id,
EXTRACT(Month from order_purchase_timestamp) As Month, EXTRACT(Year from order_purchase_timestamp) As Year
FROM `Target.Orders` AS o
LEFT JOIN `Target.Payments` AS p ON o.order_id = p.order_id)
GROUP BY payment_type, Month, Year ORDER BY Payment_type,Year,Month),
base2 as (Select *, Lag(Customer_count) over (PARTITION BY Payment_type order by Year,Month )as previous_Customer_count from base )
SELECT *, (Customer_count-previous_Customer_count)/previous_Customer_count*100 as per_INC from base2
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/0f6bcadf-a873-468e-b47f-10bfc1769bc2)


**6.2 Count of orders based on the no. of payment installments**
~~~ SQL
Select payment_installments,
Count(distinct order_id) as order_count FROM
`Target.Payments`
GROUP BY payment_installments order by order_count Desc
~~~
![image](https://github.com/neelam-ai/SQL-Data-Analysis-US-Retail-stores/assets/140748255/bca1fdfd-6b3b-4324-95b4-03113c758403)





