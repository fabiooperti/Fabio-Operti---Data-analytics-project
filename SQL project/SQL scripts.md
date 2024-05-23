# SQL scripts used in the project

### 1) Create a new database table

```sql 
CREATE TABLE food_waste(
	code INT,
	country VARCHAR(100),
	region VARCHAR(100),
	cpc_code INT,
	commodity VARCHAR(50),
	year VARCHAR(5),
	loss_percentage DECIMAL (10,2),
	loss_percentage_original VARCHAR(10),
	loss_quantity VARCHAR(100),
	activity VARCHAR(1000),
	supply_stage VARCHAR(1000),
	treatment VARCHAR(2000),
	cause_of_loss VARCHAR(2000),
	sample_size VARCHAR(2000),
	datacollection_method VARCHAR(1000),
	reference VARCHAR(1000),
	url VARCHAR(1000),
	notes VARCHAR(2000)
	)
```
### 2) Copy the CSV data to the database table

```sql
COPY food_waste
FROM 'C:\Users\Public\SQL datasets\Food loss and waste database.csv'
DELIMITER ','
CSV HEADER;
```

### 3) In order to copy the data correctly, some data types or data sizes had to be changed, for e.g.

```sql
ALTER TABLE food_waste
ALTER COLUMN region TYPE VARCHAR(1000);
```

### 4) Let’s look at the whole data

```sql
SELECT * 
FROM food_waste;
```

### 5) The dataset was exported with data from 1965 (i.e. The maximum period possible); to check all the years included in this range, I ran the following query, where I can see that the first year included is actually 1966 and it goes until 2021, with no data only for 1967 in between

```sql
SELECT DISTINCT(year)
FROM food_waste
ORDER BY year DESC;
```

### 6) I am also curious to see whether all the countries in the dataset have data points for this entire 55 years period; I query this by using the count function, requesting to count the distinct number of years that each country shows

```sql
SELECT COUNT(DISTINCT year) AS years_of_data,
       country
FROM food_waste
GROUP BY country
ORDER BY years_of_data DESC;
```

### 7) I also want to know about how many distinct commodities we have waste data available and also for each country, on how many distinct commodities we have data available about

```sql
SELECT COUNT(DISTINCT commodity)
FROM food_waste;

SELECT COUNT(DISTINCT commodity) AS distinct_commodities,
       country
FROM food_waste
GROUP BY country
ORDER BY distinct_commodities DESC;
```

### 8) Top and bottom countries by food waste

```sql
SELECT AVG(loss_percentage) AS loss_percentage_avg,
       country
FROM food_waste
GROUP BY country
ORDER BY loss_percentage_avg DESC
LIMIT 10;

SELECT AVG(loss_percentage) AS loss_percentage_avg,
       country
FROM food_waste
GROUP BY country
ORDER BY loss_percentage_avg ASC
LIMIT 10;
```

### 9) top 10 commodities and loss stages with most waste

```sql
SELECT AVG(loss_percentage) AS loss_percentage_avg,
       commodity
FROM food_waste
GROUP BY commodity
ORDER BY loss_percentage_avg DESC
LIMIT 10;

SELECT AVG(loss_percentage) AS loss_percentage_avg,
       supply_stage
FROM food_waste
GROUP BY supply_stage
ORDER BY loss_percentage_avg DESC
LIMIT 10;
```

### 10) instance of looking more in detail into some of the data queried above

```sql
SELECT *
FROM food_waste
WHERE country = 'Haiti'
```

### 11) we now ran the query for a restricted time period

```sql
WITH last_20_years AS (
SELECT * FROM food_waste
WHERE year > 2000 AND year < 2022
)
SELECT AVG(loss_percentage),
       country
FROM last_20_years
GROUP BY country
ORDER BY avg DESC;
```

### 12) let's now observe the waste trend by comparing rolling averages of the last 3 years down until the first year of available data

```sql
WITH yearly_avg AS(
SELECT AVG(loss_percentage) AS average,
       year
FROM food_waste
GROUP BY year
	)
SELECT *,
       AVG(average) OVER(
	   ORDER BY year DESC
	   ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
	   ) AS rolling_average
FROM yearly_avg
```






