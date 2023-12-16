# Step 1. Write a code to parse the data on weather in Chicago in November 2017 from the website: #
Write a code to parse the data on weather in Chicago in November 2017 from the website:
[ https://practicum-content.s3.us-west-1.amazonaws.com/data-analyst-eng/moved_chicago_weather_2017.html)
The name of the DataFrame should be weather_records, and it should be specified when you search: attrs={"id": "weather_records"} . Print the DataFrame in its entirety.
Hint
Use material from the chapter "Retrieving Data from Online Resources."

import requests
from bs4 import BeautifulSoup
import pandas as pd
URL='https://practicum-content.s3.us-west-1.amazonaws.com/data-analyst-eng/moved_chicago_weather_2017.html'
req = requests.get(URL)
soup = BeautifulSoup(req.text, 'lxml')
table = soup.find('table', attrs={"id": "weather_records"})
heading_table=[]
for row in table.find_all('th'):
    heading_table.append(row.text)   
content=[]
for row in table.find_all('tr'):
    if not row.find_all('th'):
        content.append([element.text for element in row.find_all('td')])
weather_records = pd.DataFrame(content, columns = heading_table)
print(weather_records)

You're working as an analyst for Zuber, a new ride-sharing company that's launching in Chicago. Your task is to find patterns in the available information. You want to understand passenger preferences and the impact of external factors on rides.
You'll study a database, analyze data from competitors, and test a hypothesis about the impact of weather on ride frequency. 
Description of the data
A database with info on taxi rides in Chicago:
neighborhoods table: data on city neighborhoods
	• name: name of the neighborhood
	• neighborhood_id: neighborhood code
cabs table: data on taxis
	• cab_id: vehicle code
	• vehicle_id: the vehicle's technical ID
	• company_name: the company that owns the vehicle
trips table: data on rides
	• trip_id: ride code
	• cab_id: code of the vehicle operating the ride
	• start_ts: date and time of the beginning of the ride (time rounded to the hour)
	• end_ts: date and time of the end of the ride (time rounded to the hour)
	• duration_seconds: ride duration in seconds
	• distance_miles: ride distance in miles
	• pickup_location_id: pickup neighborhood code
	• dropoff_location_id: dropoff neighborhood code
weather_records table: data on weather
	• record_id: weather record code
	• ts: record date and time (time rounded to the hour)
	• temperature: temperature when the record was taken
	• description: brief description of weather conditions, e.g. "light rain" or "scattered clouds"
Table scheme


Note: there isn't a direct connection between the tables trips and weather_records in the database. But you can still use JOIN and link them using the time the ride started (trips.start_ts) and the time the weather record was taken (weather_records.ts). 
You've already done the first part of the project: you wrote a code to parse the weather data from a website. Now you'll do the second and third parts:
Tasks 1-3: Exploratory data analysis
Tasks 4-6: Test the hypothesis that the duration of rides from the the Loop to O'Hare International Airport changes on rainy Saturdays

# Step 2: Exploratory Data Analysis #
	1. Number of Taxi Rides for Each Company (Nov 15-16, 2017)
Print the company_name field. Find the number of taxi rides for each taxi company for November 15-16, 2017, name the resulting field trips_amount, and print it, too. Sort the results by the trips_amount field in descending order.
Hint

Join the tables cabs and trips. Use aggregate functions and grouping. Don't forget to introduce a condition.
The first rows of the resulting table should look like this:
company_name	trips_amount
Flash Cab	19558
Taxi Affiliation Services	11422
Medallion Leasin	10367
Yellow Cab	9888
Taxi Affiliation Service Yellow	9299
Chicago Carriage Cab Corp	9181
City Service	8448
Sun Taxi	7701
Star North Management LLC	7455
Blue Ribbon Taxi Association Inc.	5953
Choice Taxi Association	5015

SELECT
    cabs.company_name,
    COUNT(trips.trip_id) AS trips_amount
FROM 
    cabs
    INNER JOIN 
    trips 
    ON 
    trips.cab_id = cabs.cab_id
WHERE 
    CAST(trips.start_ts AS date) BETWEEN '2017-11-15' AND '2017-11-16'
GROUP BY 
    company_name
ORDER BY 
    trips_amount DESC;

2.Number of Rides for Companies with "Yellow" or "Blue" (Nov 1-7, 2017)
Find the number of rides for every taxi company whose name contains the words "Yellow" or "Blue" for November 1-7, 2017. Name the resulting variable trips_amount. Group the results by the company_name field.
Hint

Join the tables cabs and trips. You can make calculations for one group of companies (e.g. those with the word "Yellow" in their name), then for the other, and finally group the results.
The result should look like this:
company_name	trips_amount
Taxi Affiliation Service Yellow	29213
Yellow Cab	33668
Blue Diamond	6764
Blue Ribbon Taxi Association Inc.	17675

SELECT 
    c.company_name, 
    COUNT(t.trip_id) AS trips_amount
FROM 
    cabs c
JOIN 
    trips t ON c.cab_id = t.cab_id
WHERE 
    (c.company_name LIKE '%Yellow%' OR c.company_name LIKE '%Blue%')
    AND t.start_ts >= '2017-11-01' AND t.start_ts < '2017-11-08'
GROUP BY 
    c.company_name
ORDER BY 
    trips_amount DESC;

	3 "Number of Rides for Popular Companies vs Others (November 2017)
For November 1-7, 2017, the most popular taxi companies were Flash Cab and Taxi Affiliation Services. Find the number of rides for these two companies and name the resulting variable trips_amount. Join the rides for all other companies in the group "Other." Group the data by taxi company names. Name the field with taxi company names company. Sort the result in descending order by trips_amount.
Hint

Join the tables cabs and trips. Use the CASE construction to group the companies.
The result should look like this:
company	trips_amount
Other	335771
Flash Cab	64084
Taxi Affiliation Services	37583

SELECT
    CASE
        WHEN c.company_name IN ('Flash Cab', 'Taxi Affiliation Services') THEN c.company_name
        ELSE 'Other'
    END AS company,
    COUNT(t.trip_id) AS trips_amount
FROM
    cabs c
JOIN
    trips t ON c.cab_id = t.cab_id
WHERE
    t.start_ts >= '2017-11-01' AND t.start_ts < '2017-11-08'
GROUP BY
    company
ORDER BY
    trips_amount DESC;

# Step 3: Hypothesis Testing #
## 1. Retrieving Identifiers of the O'Hare and Loop Neighborhoods ##
Retrieve the identifiers of the O'Hare and Loop neighborhoods from the neighborhoods table.
Hint

SELECT
    neighborhood_id,
    name
FROM
    neighborhoods
WHERE
    name LIKE '%Hare' OR name LIKE 'Loop';

## 2. Weather Conditions for Each Hour ##
For each hour, retrieve the weather condition records from the weather_records table. Using the CASE operator, break all hours into two groups: Bad if the description field contains the words rain or storm, and Good for others. Name the resulting field weather_conditions. The final table must include two fields: date and hour (ts) and weather_conditions.
Hint

Use the operators CASE, LIKE, and OR.
The first rows of the resulting table should look like this:
ts	weather_conditions
2017-11-01 00:00:00	Good
2017-11-01 01:00:00	Good
2017-11-01 02:00:00	Good
2017-11-01 03:00:00	Good
2017-11-01 04:00:00	Good
2017-11-01 05:00:00	Good
2017-11-01 06:00:00	Good
2017-11-01 07:00:00	Good
2017-11-01 08:00:00	Good
2017-11-01 09:00:00	Good

SELECT
    ts,
    CASE
        WHEN description LIKE '%rain%' OR description LIKE '%storm%' THEN 'Bad'
        ELSE 'Good'
    END AS weather_conditions
FROM
    weather_records;

## 3. Rides from Loop to O'Hare on Saturdays with Weather Conditions ##
Retrieve from the trips table all the rides that started in the Loop (pickup_location_id: 50) on a Saturday and ended at O'Hare (dropoff_location_id: 63). Get the weather conditions for each ride. Use the method you applied in the previous task. Also, retrieve the duration of each ride. Ignore rides for which data on weather conditions is not available.
The table columns should be in the following order:
	• start_ts
	• weather_conditions
	• duration_seconds
Sort by trip_id.
Hint

Use INNER JOIN and apply EXTRACT (DOW from trips.start_ts) = 6 to retrieve data on Saturday rides.
The first rows of the resulting table should look like this:
start_ts	weather_conditions	duration_seconds
2017-11-25 12:00:00	Good	1380.0
2017-11-25 16:00:00	Good	2410.0
2017-11-25 14:00:00	Good	1920.0
2017-11-25 12:00:00	Good	1543.0
2017-11-04 10:00:00	Good	2512.0
2017-11-11 07:00:00	Good	1440.0
2017-11-11 04:00:00	Good	1320.0
2017-11-04 16:00:00	Bad	2969.0
2017-11-18 11:00:00	Good	2280.0

SELECT
    t.start_ts,
    CASE
        WHEN w.description LIKE '%rain%' OR w.description LIKE '%storm%' THEN 'Bad'
        ELSE 'Good'
    END AS weather_conditions,
    t.duration_seconds
FROM 
    trips t
INNER JOIN 
    weather_records w ON DATE_TRUNC('hour', t.start_ts) = w.ts
WHERE 
    t.pickup_location_id = 50 AND t.dropoff_location_id = 63
    AND EXTRACT(DOW FROM t.start_ts) = 6
ORDER BY 
    t.trip_id;

# Step 4. Exploratory data analysis (Python) #
In addition to the data you retrieved in the previous tasks, you've been given a second file. You now have these two CSVs:
/datasets/project_sql_result_01.csv. It contains the following data:
company_name: taxi company name
trips_amount: the number of rides for each taxi company on November 15-16, 2017. 
/datasets/project_sql_result_04.csv. It contains the following data:
dropoff_location_name: Chicago neighborhoods where rides ended
average_trips: the average number of rides that ended in each neighborhood in November 2017. 
For these two datasets you now need to
	• import the files
	• study the data they contain
	• make sure the data types are correct
	• identify the top 10 neighborhoods in terms of drop-offs
	• make graphs: taxi companies and number of rides, top 10 neighborhoods by number of dropoffs
	• draw conclusions based on each graph and explain the results

# Step 5. Testing hypotheses (Python) #
/datasets/project_sql_result_07.csv — the result of the last query. It contains data on rides from the Loop to O'Hare International Airport. Remember, these are the table's field values:
	• start_ts
		○ pickup date and time
	• weather_conditions
		○ weather conditions at the moment the ride started
	• duration_seconds
		○ ride duration in seconds
Test the hypothesis:
"The average duration of rides from the Loop to O'Hare International Airport changes on rainy Saturdays." 
Decide where to set the significance level (alpha) on your own.
Explain:
	• how you formed the null and alternative hypotheses
	• what criterion you used to test the hypotheses and why
How will my project be evaluated?
Here are the project assessment criteria. Read them over carefully before you get to work.
Here’s what the project reviewer will look for when assessing your project:
	• how you retrieve data from the website
	• how you make data slices
	• how you group data
	• whether you use the various methods for combining data correctly
	• how you formulate hypotheses
	• what criteria you use to test the hypotheses and why
	• what conclusions you reach
	• whether you leave comments at each step
The cheat sheets and summaries from previous lessons have everything you need to complete the project.
Good luck!
