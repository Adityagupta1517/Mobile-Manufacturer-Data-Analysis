# Mobile-Manufacturer-Data-Analysis

--SQL Advance Case Study
USE master

--Q1--BEGIN 
--List all the states in which we have customers who have bought cellphones from 2005 till today.

SELECT DISTINCT B.State AS 'CUSTOMERS WHO BOUGHT CELLPHONES SINCE 2005' 
FROM FACT_TRANSACTIONS AS A
INNER JOIN DIM_LOCATION AS B
ON A.IDLocation = B.IDLocation
WHERE YEAR(A.Date)>= 2005

--Q1--END

--Q2--BEGIN
--What state in the US is buying the most 'Samsung' cell phones?

SELECT TOP 1 B.State AS 'State buying most SAMSUNG cellphones'
FROM FACT_TRANSACTIONS AS A
INNER JOIN DIM_LOCATION AS B
ON A.IDLocation = B.IDLocation
INNER JOIN DIM_MODEL AS C
ON A.IDModel = C.IDModel
INNER JOIN DIM_MANUFACTURER AS D
ON C.IDManufacturer = D.IDManufacturer
WHERE B.Country = 'US'AND D.Manufacturer_Name ='Samsung'
GROUP BY B.State
ORDER BY SUM(A.Quantity)DESC

--Q2--END

--Q3--BEGIN      
--Show the number of transactions for each model per zip code per state.

SELECT D.Country, D.State, D.ZipCode, C.Manufacturer_Name, B.Model_Name, 
COUNT(B.Model_Name) AS TRANSACTION_COUNT 
FROM FACT_TRANSACTIONS AS A
 JOIN DIM_MODEL AS B
ON A.IDModel = B.IDModel
 JOIN DIM_MANUFACTURER AS C
ON B.IDManufacturer = C.IDManufacturer
 JOIN DIM_LOCATION AS D
ON A.IDLocation = D.IDLocation
GROUP BY D.Country, D.State, D.ZipCode, C.Manufacturer_Name, B.Model_Name

--Q3--END

--Q4--BEGIN
--Show the cheapest cellphone (Output should contain the price also)

SELECT CONCAT(T.Manufacturer_Name,' ', T.Model_Name) AS Cheapest_Cellphone, T.Unit_price  
FROM ( SELECT TOP 1 B.Manufacturer_Name, A.Model_Name, A.Unit_price 
FROM DIM_MODEL AS A
INNER JOIN DIM_MANUFACTURER AS B
ON A.IDManufacturer = B.IDManufacturer
ORDER BY A.Unit_price
) AS T

--Q4--END

--Q5--BEGIN
--Find out the average price for each model in the top5 manufacturers in terms of sales quantity and order by average price.

WITH TOP_5_MANUFACTURER
AS
(SELECT TOP 5 C.Manufacturer_Name 
FROM FACT_TRANSACTIONS AS A
INNER JOIN DIM_MODEL AS B
ON A.IDModel = B.IDModel
INNER JOIN DIM_MANUFACTURER AS C
ON B.IDManufacturer = C.IDManufacturer
GROUP BY C.Manufacturer_Name
ORDER BY SUM(A.Quantity) DESC
)
SELECT C.Manufacturer_Name AS 'TOP 5 Manufacturer Name', B.Model_Name,
AVG(A.TotalPrice) AS AVERAGE_PRICE 
FROM FACT_TRANSACTIONS AS A
INNER JOIN DIM_MODEL AS B
ON A.IDModel = B.IDModel
INNER JOIN DIM_MANUFACTURER AS C
ON B.IDManufacturer = C.IDManufacturer
WHERE C.Manufacturer_Name IN (SELECT * FROM TOP_5_MANUFACTURER)
GROUP BY C.Manufacturer_Name, B.Model_Name
ORDER BY AVERAGE_PRICE DESC

--Q5--END

--Q6--BEGIN
--List the names of the customers and the average amount spent in 2009, where the average is higher than 500

SELECT A.Customer_Name, AVG(B.TotalPrice)
AS AVG_AMT_MORETHAN_500_SPENT_IN_2009 
FROM DIM_CUSTOMER AS A
INNER JOIN FACT_TRANSACTIONS AS B
ON A.IDCustomer = B.IDCustomer
WHERE YEAR(B.Date) = 2009
GROUP BY A.Customer_Name
HAVING AVG(B.TotalPrice)>500
ORDER BY AVG(B.TotalPrice) DESC




--Q6--END
	
--Q7--BEGIN  
--List if there is any model that was in the top 5 in terms of quantity, simultaneously in 2008, 2009 and 2010 

SELECT Model_Name from (
SELECT TOP 5 
    B.Model_Name, 
    SUM(A.Quantity) AS TOTAL_QTY
	from FACT_TRANSACTIONS as A
JOIN DIM_MODEL AS B ON A.IDModel = B.IDModel
JOIN DIM_MANUFACTURER AS C ON B.IDManufacturer = C.IDManufacturer

WHERE YEAR(A.Date) = 2008
GROUP BY 
    C.Manufacturer_Name, 
    B.Model_Name, 
    A.IDModel
ORDER BY TOTAL_QTY DESC
) as X

INTERSECT

SELECT Model_Name from (
SELECT TOP 5 
    B.Model_Name, 
    SUM(A.Quantity) AS TOTAL_QTY
	from FACT_TRANSACTIONS as A
JOIN DIM_MODEL AS B ON A.IDModel = B.IDModel
JOIN DIM_MANUFACTURER AS C ON B.IDManufacturer = C.IDManufacturer

WHERE YEAR(A.Date) = 2009
GROUP BY 
    C.Manufacturer_Name, 
    B.Model_Name, 
    A.IDModel
ORDER BY TOTAL_QTY DESC
) as X

INTERSECT

SELECT Model_Name from (
SELECT TOP 5
    B.Model_Name, 
    SUM(A.Quantity) AS TOTAL_QTY
	from FACT_TRANSACTIONS as A
JOIN DIM_MODEL AS B ON A.IDModel = B.IDModel
JOIN DIM_MANUFACTURER AS C ON B.IDManufacturer = C.IDManufacturer

WHERE YEAR(A.Date) = 2010
GROUP BY 
    C.Manufacturer_Name, 
    B.Model_Name, 
    A.IDModel
ORDER BY TOTAL_QTY DESC
) as X



--Q7--END	
--Q8--BEGIN
--. Show the manufacturer with the 2nd top sales in the year of 2009 and
--the manufacturer with the 2nd top sales in the year of 2010.

SELECT T1.SALE_YEAR, T1.Manufacturer_Name as '2nd TOP MANUFACTURER IN SALES', T1.TOTAL_SALES 
FROM
(
SELECT YEAR(T.Date) AS SALE_YEAR, T.Manufacturer_Name, SUM(T.TotalPrice) AS TOTAL_SALES,
DENSE_RANK () OVER(PARTITION BY YEAR(T.Date) ORDER BY SUM(T.TotalPrice) DESC) AS RANK_
FROM
(SELECT A.*, C.Manufacturer_Name
FROM FACT_TRANSACTIONS AS A
 JOIN DIM_MODEL AS B
 ON A.IDModel = B.IDModel
 JOIN DIM_MANUFACTURER AS C
 ON B.IDManufacturer = C.IDManufacturer
 ) AS T
  WHERE YEAR(T.Date) IN ('2009', '2010')
  GROUP BY T.Manufacturer_Name, YEAR(T.Date)
) AS T1
WHERE T1.RANK_= 2

--Q8--END
--Q9--BEGIN
--. Show the manufacturers that sold cellphones in 2010 but did not in 2009.

SELECT A.Manufacturer_Name AS 'MANUFACTURER WHO SOLD CELL PHONES IN IN 2010 BUT NOT IN 2009' 
FROM DIM_MANUFACTURER AS A
JOIN DIM_MODEL AS B
ON A.IDManufacturer = B.IDManufacturer
JOIN FACT_TRANSACTIONS AS C
ON B.IDModel = C.IDModel
WHERE YEAR (C.Date) = 2010
EXCEPT 
SELECT A.Manufacturer_Name FROM DIM_MANUFACTURER AS A
JOIN DIM_MODEL AS B
ON A.IDManufacturer = B.IDManufacturer
JOIN FACT_TRANSACTIONS AS C
ON B.IDModel = C.IDModel
WHERE YEAR (C.Date) = 2009

--Q9--END

--Q10--BEGIN
-- Find top 10 customers and their average spend, average quantity by each year
--Also find the percentage of change in their spend.		
SELECT top 10 T1.IDCustomer, B.Customer_Name, T1.SALE_YEAR, T1.TOTAL_SPEND, T1.AVG_QTY, T1.AVG_SPEND,
CONCAT(((T1.TOTAL_SPEND - T1.LAG_)/T1.LAG_)* 100,' %') AS PERCENT_GROWTH
FROM
(
 SELECT *,
 LAG(T.TOTAL_SPEND, 1) OVER(PARTITION BY T.IDCUSTOMER ORDER BY T.SALE_YEAR) AS LAG_
 FROM
(
 SELECT A.IDCustomer, YEAR(A.Date) AS SALE_YEAR, AVG(A.TotalPrice) AS AVG_SPEND, 
 AVG(A.Quantity) AS AVG_QTY, SUM(A.TotalPrice) AS TOTAL_SPEND
 FROM FACT_TRANSACTIONS AS A
 WHERE A.IDCustomer IN 
( SELECT TOP 10 P.IDCustomer FROM FACT_TRANSACTIONS AS P
  GROUP BY P.IDCustomer ORDER BY SUM(P.TotalPrice) DESC
) 
  GROUP BY YEAR(A.Date), A.IDCustomer
) AS T
) AS T1
JOIN DIM_CUSTOMER AS B
ON T1.IDCustomer = B.IDCustomer

--Q10--END
	
