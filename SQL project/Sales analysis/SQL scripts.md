# SQL scripts used in the analysis


### Let's keep track of all the orders that we received, ordered from the last one we received
```sql
SELECT * FROM Orders
ORDER BY CreationDate DESC;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/6da00181-1de3-4c46-bd9c-c2cf126f4d8c)

### Can we add a month column next to it?
```
SELECT *,
       MONTH(CreationDate) AS Month,
       MONTHNAME(CreationDate) AS MonthName
FROM Orders
ORDER BY CreationDate DESC;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/85b5af64-9e8c-41c1-8ee8-1857adfed3ef)

### Let's dive into the numbers - I want to see each month orders (n. of orders, quantities sold and amounts) in chronological order
```
SELECT YEAR(a.CreationDate) AS OrderYear,
       MONTHNAME(a.CreationDate) AS OrderMonth,
       COUNT(a.OrderID) AS N_orders,
       SUM(b.Quantity) AS Quantity_sold,
       SUM(DISTINCT a.TotalDue) AS Sales
FROM Orders a
LEFT OUTER JOIN OrderItem b
ON a.OrderID = b.OrderID
GROUP BY 2,1
ORDER BY OrderYear, MONTH(a.CreationDate)
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/394023b9-ea58-4934-a244-9c01b93df37a)

### Who are our top customers? Let's order orders by customer from the largest
```
SELECT FirstName,
       LastName,
       SUM(TotalDue) AS Tot_Revenue      
FROM Orders AS a
LEFT OUTER JOIN Customer AS b
ON a.CustomerID = b.CustomerID
GROUP BY 1,2
ORDER BY Tot_Revenue DESC;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/d1ed5887-1559-4489-b65c-5bda5f198839)
### There are 178 customers that have ordered from us so far

### Who are the customers that ordered from us only once?
```
SELECT Customer.CustomerID,
       FirstName,
       LastName,
       COUNT(DISTINCT Orders.OrderID) N_Orders
FROM Orders
LEFT OUTER JOIN Customer 
ON Orders.CustomerID = Customer.CustomerID
GROUP BY 1
HAVING COUNT(DISTINCT Orders.OrderID) = 1;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/488e4455-833f-4aa3-a4a6-ed49658c80cc)

### It turns out it's most of the customers - 158 out of 178
### So what's our customers repeat rate (i.e. customers that ordered from us at least twice) ?
We create a CTE (common table expression), i.e. a temporary query used in the context of a larger query
In this case the CTE is the table of customers that ordered from us more than once
and we join this with the Orders table where we can get the total number of distinct customers that have ordered at least once from us
```
WITH Repeat_customers AS 
(
  SELECT CustomerID AS Repeat_Cust
  FROM Orders
  GROUP BY 1
  HAVING COUNT(OrderID) > 1
)
SELECT (COUNT(DISTINCT Repeat_Cust)/COUNT(DISTINCT CustomerID))*100
FROM Orders
LEFT OUTER JOIN Repeat_customers
ON Orders.CustomerID = Repeat_customers.Repeat_Cust
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/be838a1d-f065-4f63-8ae8-d72bf4b66945)

### Can we see all the customers that have never ordered from us in our database? We can still try to convert them maybe...
```
SELECT
FirstName,
LastName,
COUNT(OrderID) as TotalOrders
FROM Customer
LEFT OUTER JOIN Orders
ON Customer.CustomerID = Orders.CustomerID 
GROUP BY Customer.CustomerID
HAVING COUNT(OrderID) = 0;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/96453ddb-9d31-4ce7-bcaf-560bb554d9ca)

It's 822 customers! It turns out the majority of the customers in our database have never eventually ordered from us yet!

### We want to run a marketing campaign to convert some of these customers. We need their contact data. Let's query the customer table
### and find out whether we have any missing data
```
SELECT *
FROM Customer
WHERE FirstName IS NULL OR
LastName IS NULL OR
Email IS NULL OR
Phone IS NULL
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/f07eab51-40e7-4d04-ade1-27b14599c6f2)

### We see some customers missing email or phone data, let's exclude these from our campaign list
```
SELECT FirstName,
       LastName,
       Email,
       Phone
FROM Customer
WHERE Email IS NOT NULL AND
      Phone IS NOT NULL;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/56be4037-8f70-4871-a9fb-400ec6d45c00)


### Let's focus on the products - How many unique products have we sold so far and in which quantities?
```
SELECT COUNT(DISTINCT ProductID) AS TotUniqueProducts,
       SUM(Quantity) AS TotQuantitySold
FROM OrderItem;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/4ec32f68-01d7-4cf1-a0ee-ba59a7415f0f)

### Which have been our top 3 products?
```
SELECT Variety,
       Size,
       SUM(OrderItem.Quantity) as QuantitySold
FROM Product
RIGHT OUTER JOIN OrderItem
ON Product.ProductID = OrderItem.ProductID
GROUP BY Product.ProductID
ORDER BY QuantitySold DESC
LIMIT 3;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/3da891da-e9f9-45d7-903d-61139beaf3b7)

### We sell our products in two different sizes - what's the most popular size?
```
SELECT Product.Size,
       SUM(OrderItem.Quantity) as QuantitySold
FROM Product
RIGHT OUTER JOIN OrderItem
ON Product.ProductID = OrderItem.ProductID
GROUP BY Product.Size
ORDER BY QuantitySold DESC;
```
![image](https://github.com/fabiooperti/Fabio-Operti---Data-analytics-project/assets/170554271/8029fd53-a676-4d8b-95cb-24f82e6b5124)








