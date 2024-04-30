# Chicago Crime & Weather
## Questions and Answers

**1.**  List the total number of reported crimes between 2018 and 2023?

````sql
SELECT 
	to_char(count(*), '9g999g999') AS "Total Reported Crimes"
FROM 
	chicago.crimes;
````

**Results:**

Total Reported Crimes|
---------------------|
 1,450,979           |

**2.** List the total amount of Homicides, Batteries and Assaults reported between 2018 and 2023.

````sql
SELECT 
	initcap(crime_type) AS crime_type,
	count(*) AS n_crimes
FROM 
	chicago.crimes
WHERE 
	crime_type IN ('homicide', 'battery', 'assault')
GROUP BY 
	crime_type
ORDER BY 
	n_crimes DESC;
````

**Results:**

crime_type|n_crimes|
----------|--------|
Battery   |  266357|
Assault   |  122997|
Homicide  |    4066|

**3.** Which are the 3 most common crimes reported and what percentage amount are they from the total amount of reported crimes?

```sql
WITH get_top_crime AS (
	SELECT 
		initcap(crime_type) AS crime_type,
		count(*) AS n_crimes
	FROM 
		chicago.crimes
	GROUP BY 
		crime_type
	ORDER BY 
		n_crimes DESC
)
SELECT
	crime_type,
	n_crimes,
	round(100 * n_crimes::NUMERIC / sum(n_crimes) OVER (), 2) AS total_percentage
FROM
	get_top_crime
LIMIT 3;
```

**Results:**

crime_type     |n_crimes|total_percentage|
---------------|--------|----------------|
Theft          |  321957|           22.19|
Battery        |  266357|           18.36|
Criminal Damage|  161766|           11.15|

**4.** What are the top ten communities that had the MOST amount of crimes reported?  Include the current population, density and order by the number of reported crimes.

````sql
SELECT 
	initcap(t2.community_name) AS community,
	t2.population,
	t2.density,
	count(*) AS reported_crimes
FROM 
	chicago.crimes AS t1
JOIN
	chicago.community AS t2
ON 
	t2.community_id = t1.community_id
GROUP BY 
	t2.community_name,
	t2.population,
	t2.density
ORDER BY 
	reported_crimes DESC
LIMIT 10;
````

**Results:**

community             |population|density |reported_crimes|
----------------------|----------|--------|---------------|
Austin                |     96557|13504.48|          79271|
Near North Side       |    105481|38496.72|          63084|
Near West Side        |     67881|11929.88|          52091|
South Shore           |     53971|18420.14|          49722|
Loop                  |     42298|25635.15|          49003|
North Lawndale        |     34794|10839.25|          46155|
Humboldt Park         |     54165|15045.83|          41949|
West Town             |     87781|19166.16|          40772|
Auburn Gresham        |     44878|11903.98|          40514|
Greater Grand Crossing|     31471| 8865.07|          37429|

**5.** What are the top ten communities that had the LEAST amount of crimes reported?  Include the current population, density and order by the number of reported crimes.

````sql
SELECT 
	initcap(t2.community_name) AS community,
	t2.population,
	t2.density,
	count(*) AS reported_crimes
FROM 
	chicago.crimes AS t1
JOIN
	chicago.community AS t2
ON 
	t2.community_id = t1.community_id
GROUP BY 
	t2.community_name,
	t2.population,
	t2.density
ORDER BY 
	reported_crimes
LIMIT 10;
````

**Results:**

community      |population|density |reported_crimes|
---------------|----------|--------|---------------|
Edison Park    |     11525|10199.12|           1623|
Burnside       |      2527| 4142.62|           2129|
Forest Glen    |     19596| 6123.75|           3135|
Mount Greenwood|     18628|  6873.8|           3150|
Montclare      |     14401|14546.46|           3616|
Hegewisch      |     10027| 1913.55|           3632|
Oakland        |      6799|11722.41|           4267|
Fuller Park    |      2567| 3615.49|           4342|
Archer Heights |     14196| 7062.69|           5036|
Mckinley Park  |     15923|11292.91|           5048|

**6.** What month had the most crimes reported and what was the average and median temperature high in the last six years?

````sql
ELECT
	to_char(t1.reported_crime_date, 'Month') AS month,
	COUNT(*) AS n_crimes,
	round(avg(t2.temp_high), 1) avg_high_temp,
	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY t2.temp_high) AS median_high_temp
FROM
	chicago.crimes AS t1
JOIN 
	chicago.weather AS t2
ON 
	t1.reported_crime_date = t2.weather_date
GROUP BY
	month
ORDER BY
	n_crimes DESC;
````

**Results:**

month    |n_crimes|avg_high_temp|median_high_temp|
---------|--------|-------------|----------------|
July     |  135240|         85.1|            86.0|
August   |  134712|         84.1|            85.0|
October  |  128470|         62.9|            62.0|
June     |  127774|         81.6|            81.0|
September|  127567|         77.4|            78.0|
May      |  126130|         72.2|            73.0|
December |  117531|         41.3|            41.0|
November |  116688|         48.3|            47.0|
March    |  113630|         47.7|            47.0|
January  |  113133|         33.4|            34.0|
April    |  109391|         57.8|            56.0|
February |  100713|         36.6|            37.0|

**7.** What month had the most homicides reported and what was the average and median temperature high in the last six years?

````sql
SELECT
	to_char(t1.reported_crime_date, 'Month') AS month,
	COUNT(*) AS n_crimes,
	round(avg(t2.temp_high), 1) avg_high_temp,
	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY t2.temp_high) AS median_high_temp
FROM
	chicago.crimes AS t1
JOIN 
	chicago.weather AS t2
ON 
	t1.reported_crime_date = t2.weather_date
WHERE
	t1.crime_type = 'homicide'
GROUP BY
	month
ORDER BY
	n_crimes DESC;
````

**Results:**

month    |n_crimes|avg_high_temp|median_high_temp|
---------|--------|-------------|----------------|
July     |     459|         85.3|            86.0|
June     |     432|         82.5|            83.0|
September|     404|         78.1|            79.0|
May      |     391|         73.1|            74.0|
August   |     389|         84.4|            85.0|
October  |     350|         63.9|            64.0|
April    |     331|         59.8|            58.0|
November |     317|         50.2|            49.0|
December |     287|         41.7|            42.0|
January  |     256|         33.8|            34.0|
February |     228|         36.5|            37.5|
March    |     222|         49.1|            48.0|

**8.** List the most violent year and the number of arrests with percentage.  Order by the number of crimes in decending order.  Determine the most violent year by the number of reported Homicides, Assaults and Battery for that year.

````sql
WITH get_arrest_percentage AS (
	SELECT
		EXTRACT('year' FROM t1.reported_crime_date) AS most_violent_year,
		count(*) AS reported_violent_crimes,
		sum(
			CASE
				WHEN arrest = TRUE THEN 1
				ELSE 0
			END 
		) AS number_of_arrests
	FROM
		chicago.crimes AS t1
	WHERE 
		crime_type IN ('homicide', 'battery', 'assault')
	GROUP BY
		most_violent_year
	ORDER BY
		reported_violent_crimes DESC
)
SELECT
	most_violent_year,
	reported_violent_crimes,
	number_of_arrests || ' (' || round(100 * number_of_arrests::NUMERIC / reported_violent_crimes, 2) || '%)' AS number_of_arrests
FROM
	get_arrest_percentage;
````

**Results:**

most_violent_year|reported_violent_crimes|number_of_arrests|
-----------------|-----------------------|-----------------|
2018|                  70835|13907 (19.63%)   |
2019|                  70645|14334 (20.29%)   |
2023|                  67355|9340 (13.87%)    |
2022|                  62412|8165 (13.08%)    |
2021|                  61611|7855 (12.75%)    |
2020|                  60562|9577 (15.81%)    |

**9.** List the day of the week, year, average precipitation, average high temperature and the highest number of reported crimes for days with and without precipitation.

````sql
DROP TABLE IF EXISTS weekday_precipitation_values;
CREATE TEMP TABLE weekday_precipitation_values AS (
	SELECT
		EXTRACT('year' FROM t1.reported_crime_date) AS crime_year,
		to_char(t1.reported_crime_date, 'Day') AS day_of_week,
		round(avg(t2.precipitation)::NUMERIC, 2) AS avg_precipitation,
		round(avg(t2.temp_high)::NUMERIC, 2) AS avg_temp_high,
		COUNT(*) AS n_crimes,
		DENSE_RANK() OVER(PARTITION BY EXTRACT('year' FROM t1.reported_crime_date) ORDER BY count(*) DESC) AS rnk
	FROM
		chicago.crimes AS t1
	JOIN 
		chicago.weather AS t2
	ON
		t1.reported_crime_date = t2.weather_date
	WHERE
		t2.precipitation > 0
	GROUP BY
		crime_year,
		day_of_week
	ORDER BY
		n_crimes DESC
);

DROP TABLE IF EXISTS weekday_values;
CREATE TEMP TABLE weekday_values AS (
	SELECT
		EXTRACT('year' FROM t1.reported_crime_date) AS crime_year,
		to_char(t1.reported_crime_date, 'Day') AS day_of_week,
		round(avg(t2.precipitation)::NUMERIC, 2) AS avg_precipitation,
		round(avg(t2.temp_high)::NUMERIC, 2) AS avg_temp_high,
		COUNT(*) AS n_crimes,
		DENSE_RANK() OVER(PARTITION BY EXTRACT('year' FROM t1.reported_crime_date) ORDER BY count(*) DESC) AS rnk
	FROM
		chicago.crimes AS t1
	JOIN 
		chicago.weather AS t2
	ON
		t1.reported_crime_date = t2.weather_date
	WHERE
		t2.precipitation = 0
	GROUP BY
		crime_year,
		day_of_week
	ORDER BY
		n_crimes DESC
);

SELECT
	t1.crime_year,
	t1.day_of_week,
	t1.avg_precipitation,
	t1.avg_temp_high,
	t1.n_crimes,
	t2.day_of_week,
	t2.avg_temp_high,
	t2.n_crimes,
	t2.n_crimes - t1.n_crimes AS n_crime_difference
FROM
	weekday_precipitation_values AS t1
JOIN
	weekday_values AS t2
ON
	t1.crime_year = t2.crime_year
WHERE
	t1.rnk = 1
AND
	t2.rnk = 1
ORDER BY
	t1.crime_year;
````

**Results:**

crime_year|day_of_week|avg_precipitation|avg_temp_high|n_crimes|day_of_week|avg_temp_high|n_crimes|n_crime_difference|
----------|-----------|-----------------|-------------|--------|-----------|-------------|--------|------------------|
2018|Monday     |             0.57|        60.23|   16455|Friday     |        60.24|   25667|              9212|
2019|Wednesday  |             0.31|        64.00|   19411|Friday     |        57.09|   27042|              7631|
2020|Wednesday  |             0.29|        63.54|   11394|Sunday     |        61.51|   23477|             12083|
2021|Sunday     |             0.30|        57.12|   10889|Monday     |        65.31|   22840|             11951|
2022|Friday     |             0.21|        51.44|   13029|Monday     |        59.95|   26026|             12997|
2023|Friday     |             0.24|        60.14|   15855|Tuesday    |        65.40|   27157|             11302|

**10.** List the days with the most reported crimes when there is zero precipitation and the day when precipitation is greater than .5".  Include the day of the week, high temperature, amount and precipitation and the total number of reported crimes for that day.

````sql
WITH precipitation_false AS (
	SELECT
		t1.reported_crime_date,
		to_char(t1.reported_crime_date, 'Day') AS day_of_week,
		t2.temp_high,
		t2.precipitation,
		count(*) AS reported_crimes
	FROM
		chicago.crimes AS t1
	JOIN
		chicago.weather AS t2
	ON
		t1.reported_crime_date = t2.weather_date
	WHERE
		t2.precipitation = 0
	GROUP BY 
		day_of_week,
		t2.precipitation,
		temp_high,
		t1.reported_crime_date
	ORDER BY
		reported_crimes DESC
	LIMIT 
		1
),
precipitation_true AS (
	SELECT
		t1.reported_crime_date,
		to_char(t1.reported_crime_date, 'Day') AS day_of_week,
		t2.temp_high,
		t2.precipitation,
		count(*) AS reported_crimes
	FROM
		chicago.crimes AS t1
	JOIN
		chicago.weather AS t2
	ON
		t1.reported_crime_date = t2.weather_date
	WHERE
		t2.precipitation > .5
	GROUP BY 
		day_of_week,
		temp_high,
		t2.precipitation,
		t1.reported_crime_date
	ORDER BY
		reported_crimes DESC
	LIMIT 
		1
)
SELECT
	*
FROM 
	precipitation_false
UNION
SELECT
	*
FROM 
	precipitation_true
ORDER BY
	reported_crimes DESC;
````

**Results:**

reported_crime_date|day_of_week|temp_high|precipitation|reported_crimes|
-------------------|-----------|---------|-------------|---------------|
2020-05-31|Sunday     |       69|          0.0|           1899|
2018-10-01|Monday     |       72|         1.56|            926|

To be continued....





