Q1.
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey

![Screenshot 2024-06-19 155828](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/801c990d-d9f5-4139-89a1-150e623d5061)

Observing these 8 coustomers Journey

Customer_ID 1 start free trial on 1/Aug/2020 and has jumped to base monthly on Dt 8/Aug/2020 (with limited access only

Customer_Id 3 , customer_id 14  also same as customer_id 1

Customer_id 23 start free trial on 13/05/2020 and has jumped to pro annual yearly plan on Dt 20/05/2020

Customer_id 34 start free trial on 20/12/2020 after 7 days he joined monthly basic plane ater 2 month they
join pro monthly plan on Dt 26/03/2021

Customer_id 42 do as same as 34 

customer_id 56 free trial start on 03/01/2020 then they directly join pro anuual plan on Dt 10/01/2020

Customer_id 62 start free trial on Dt 12/10/2020
on Dt 19/10/2020 they buy basic monthly
& on Dec/2020 he did not have anu plan
Dt 02/01/2021 they buy pro monthly 
and after 23/02/2021 they did not buy any plan , the customer is lost 

1. All customers started with a 7-day free trial and upgraded their subscription plan.

2. All customers satisfied with Foodie-Fi content because they always upgraded their free trial account
 into ‘paid’ membership.

3.Only Customer_id 62 not upgrade there plane after 2 month of subscriptions

4. Vast majority of the subscribed customers upgraded their plan after a while

Q2.How many customers has Foodie-Fi ever had?
![Screenshot 2024-06-19 160435](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/a0317b90-6b1c-43c0-a86b-d7db7712c5ee)

Q3 .What is the monthly distribution of trial plan start_date values for our dataset -
use the start of the month as the group by value

![Screenshot 2024-06-19 160644](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/19647911-5c33-4edb-a8f0-45178fb7f9f0)

Insight:-
We can see that March has the highest number of trial plans

where as February has the lowest number of trial plans.

Q4. What plan start_date values occur after the year 2020 for our dataset?
— Show the breakdown by count of events for each plan_name

SELECT 
p.plan_id,p.plan_name,COUNT(customer_id) AS count_customer
FROM plans AS p
JOIN subscriptions AS s
ON p.plan_id=s.plan_id
WHERE YEAR(s.start_date)>2020
GROUP BY 
p.plan_id,p.plan_name


SELECT 
p.plan_id,p.plan_name,COUNT(customer_id) AS count_customer
FROM plans AS p
JOIN subscriptions AS s
ON p.plan_id=s.plan_id
WHERE 
YEAR(start_date) <=2021
GROUP BY 
p.plan_id,p.plan_name

/*Insight:- We can see that after 2020,
pro_annual and pro_monthly plans are sold the most. 
It indicates that people are really
liking the services as 2020 was the starting year for the company,
so they are satisfied with the services.*/

![Screenshot 2024-06-19 160923](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/c9713bf9-8c42-4857-9fd7-6d0366edb6e4)

![Screenshot 2024-06-19 161055](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/23a26189-4875-43a9-a2f4-0f56154f7f59)

But from Above analysis we see that count of customers in each plan is
less in 2021 as compared to 2020.
Also, there were 0 customer on trial plan in 2021.
Does it mean that there were no new customers in 2021, 
or did they jump directly on basic monthly or other plans 
without going through the trial? 

Q5.What is the customer count and percentage of customers who have 
churned rounded to 1 decimal place?*/

-- Chrun Customer %= Chrun Customer/total customer
DECLARE 
@churn INT,
@total_customer INT

SET @churn= 
(SELECT COUNT(DISTINCT customer_id)
FROM 
subscriptions
WHERE plan_id=4);

SET @total_customer=
(SELECT COUNT(DISTINCT customer_id) 
FROM subscriptions)

SELECT @churn as [Churned Customers], @total_customer as [Total Customers],
CAST(@churn / cast(@total_customer as decimal(10,1)) *100 AS DECIMAL(10, 1)) AS Churned_Percentage;

-- Alternate methode

SELECT COUNT(DISTINCT CASE WHEN plan_id =4 THEN customer_id END) AS [chruned customer],
COUNT(DISTINCT customer_id ) AS [total_customer],
CAST((COUNT(DISTINCT CASE WHEN plan_id =4 THEN customer_id END)/
CAST(COUNT(DISTINCT customer_id )AS decimal(10,1))) AS DECIMAL(10,1))*100 

FROM subscriptions

![Screenshot 2024-06-19 161537](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/0be0908d-87d4-4a8d-8aa0-288a87bc601b)

Insights: There are 307 customers who churned, which is 30.7% of Foodie-Fi customer base.

Q6.How many customers have churned straight after their initial free trial


WITH cte AS
(
	SELECT customer_id,plan_id,ROW_NUMBER() OVER(PARTITION BY customer_id ORDER By start_date)AS rn
	FROM
	subscriptions)

SELECT 
COUNT(DISTINCT CASE WHEN plan_id=4 AND rn=2 THEN customer_id END)AS [churn customer immediatly after basic trial plan],
CAST((COUNT(DISTINCT CASE WHEN plan_id=4 AND rn=2 THEN customer_id END)/
CAST(COUNT(DISTINCT customer_id)AS DECIMAL(10,1)))*100 AS DECIMAL(10,1))
AS [Churn %]
from cte
;

![Screenshot 2024-06-19 161838](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/b5b319c7-d721-4609-a097-c275ad602e78)


-- Insight:-
-- 92 customers churned straight after the initial free trial which is 9% of entire customer base.
--out of 1000 customers who have taken trial plan, only 92 did not upgrade to any plan

Q7.What is the number and percentage of customer plans after their initial free trial?

Understanding: Question is asking for number and percentage of customers who converted to becoming paid customer after the trial.

Logic Used:

Find customer’s next plan which is located in the next row using LEAD(). Run the next_plan_cte separately to
view the next plan results and understand how LEAD() works.
Filter for plan_id = 0 as every customer has to start from the trial plan at 0.
Then we’ll group by next_plan_id and count the customers for each plan_id


WITH next_plan_cte AS(
		SELECT customer_id,plan_id,
		LEAD(plan_id) OVER(PARTITION BY customer_id ORDER BY start_date) AS [next plan after free trial]
		FROM subscriptions
	),
cte1 AS
(SELECT * FROM	next_plan_cte 
  WHERE plan_id=0),

cte2 AS
(
  SELECT [next plan after free trial] AS [Next Plan ID after Basic Trial] ,
  COUNT(DISTINCT customer_id) AS [count of customer]
  FROM cte1 
  GROUP BY 
  [next plan after free trial]
  )
,
		cte3 AS
		(
		SELECT *,
		SUM(CAST([count of customer] AS int)) over() AS [total Customer]
		FROM cte2
		)
SELECT [Next Plan ID after Basic Trial],
[count of customer],
cast(([count of customer]/cast([total Customer] AS decimal(10,1))*100) As decimal(10,1)) AS [% of customer each plan]
FROM cte3

![Screenshot 2024-06-19 162157](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/968d7933-8d41-49a4-8898-3adec38436ff)


Insight:-
More than 80% of customers are on paid plans but a small 3.7% on plan 3 (pro annual $199).
Foodie-Fi has to strategize on their customer acquisition who would be willing to spend more.
There is a good point then only 9% is the churn after free trial, we can still try to reduce it.


 Q7. How many customers have upgraded to an annual plan in 2020?
-- here we just have to count number of customers for plan_id = 3 and year of start date should be 2020 only

select count(*) as upgraded_customers_to_annual_plan
from subscriptions
where plan_id = 3 
and year(start_date) = 2020

![Screenshot 2024-06-19 162338](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/0d0493f0-7651-44d6-9977-bdb1d6dced33)

How many days on average does it take for a customer to an annual plan from the 
day they join Foodie-Fi?

![Screenshot 2024-06-19 162522](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/92a89b28-d85e-4fc2-aeee-d0ba40e3efa3)

Can you further breakdown this average value into 30 day periods (i.e. 0–30 days, 31–60 days etc)

WITH 
CTE AS(
SELECT distinct customer_id,MIN(start_date) over(partition by customer_id
order by start_date) AS First_start_date,
case when plan_id=3 then start_date end AS upgrade_date
from subscriptions) ,

CTE1 AS(
			SELECT *,DATEDIFF(DAY,First_start_date,upgrade_date) AS
			DAYS_taken_to_convert
			from CTE
			WHERE DATEDIFF(DAY,First_start_date,upgrade_date) is not null)
-- SELECT * FROM CTE1
,
CTE2 AS(
SELECT 
* ,
case when DAYS_taken_to_convert >=0 and days_taken_to_convert < 30 then '0–29 days'
 when DAYS_taken_to_convert >=30 and days_taken_to_convert < 60 then '30–59 days'
 when DAYS_taken_to_convert >=60 and days_taken_to_convert < 90 then '60–89 days'
 when DAYS_taken_to_convert >=90 and days_taken_to_convert < 120 then '90–119 days'
 when DAYS_taken_to_convert >=120 and days_taken_to_convert < 150 then '120-149 days'
 when DAYS_taken_to_convert >=150 and days_taken_to_convert < 180 then '150–179 days'
 when DAYS_taken_to_convert >=180 and days_taken_to_convert < 210 then '180–209 days'
 when DAYS_taken_to_convert >=210 and days_taken_to_convert < 240 then '210–239 days'
 when DAYS_taken_to_convert >=240 and days_taken_to_convert < 270 then '240–269 days'
 when DAYS_taken_to_convert >=270  then '271 and above days'
end AS period_
FROM CTE1)
,

CTE3 AS(
SELECT *,
case when period_='0–29 days' then 1
when period_='30–59 days' then 2
when period_='60–89 days' then 3
when period_='90–119 days' then 4
when period_='120-149 days' then 5
when period_='150–179 days' then 6
when period_='180–209 days' then 7
when period_='210–239 days' then 8
when period_='240–269 days' then 9
when period_='271 and above days' then 10
end AS period_g
from CTE2
)

SELECT period_,COUNT(period_g) AS [total customer]
from CTE3
group by 
period_
Order by COUNT(period_g) 

![Screenshot 2024-06-19 162714](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/f988f224-0aa3-4621-9e0b-edad7209af87)


 What is the customer count and percentage breakdown of all 5 plan_name values at 2020–12–31

 WITH cte AS(
SELECT 
s.customer_id,s.start_date,p.plan_name,
case
when plan_name='trial' then DATEADD(day,7,start_date)
when plan_name='basic monthly' then DATEADD(MONTH,1,start_date)
when plan_name='pro monthly' then DATEADD(MONTH,1,start_date)
when plan_name='pro annual' then DATEADD(YEAR,1,start_date)
end AS end_date
from subscriptions as s
join plans as p
on s.plan_id=p.plan_id
),
cte1 as (
SELECT * FROM 
cte
where start_date<='2020-12-31' AND end_date<'2021-12-31'
),
cte2 as(
SELECT plan_name,COUNT(customer_id) AS Count_of_customer
from 
cte1
group by plan_name)
SELECT plan_name,Count_of_customer,
CAST((Count_of_customer/cast(sum(Count_of_customer)over()as decimal(10,1)))*100 AS decimal(10,2))
AS pct_customer
from cte2

![Screenshot 2024-06-19 162844](https://github.com/Ankur-dahake/Foodie-Fi/assets/127744701/67250350-4ec5-4e13-8f44-306657c6c5af)








