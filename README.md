# Cyclistic Bike-Share Analysis: Maximizing Annual Memberships
![cylistic](https://github.com/thejagadeesh/Cyclistic-bike-share-analysis/assets/114074976/16142766-bcca-4215-8d2c-9658cb7defb4)

## Introduction:
Welcome to the Cyclistic bike-share analysis case study! As a junior data analyst in the marketing team at Cyclistic, a bike-share company in Chicago, my primary objective was to maximize annual memberships by understanding how casual riders and annual members utilize Cyclistic bikes differently. This case study presents the data analysis process, insights, and actionable recommendations to boost annual memberships through targeted marketing strategies.

## Business Task:
My manager, Lily Moreno, assigned me the task of analyzing the usage patterns of annual members and casual riders to identify opportunities to increase annual memberships. The goal was to leverage data insights to design a compelling marketing program and drive growth in the number of annual members.

## Data Sources:
To conduct the analysis, I utilized real bike-share data from Motivate International Inc., treated as Cyclistic's company data. The dataset included 12 CSV files covering the period from February 2022 to January 2023. The essential columns included ride_id, rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id, start_lat, start_lng, end_lat, end_lng, and member_casual.

## Data Preparation and Cleaning:
Due to the massive size of the data (over 5 million rows), I shifted from Excel to SQL in BigQuery for efficient processing. The initial steps involved combining the CSV files using the UNION operator and performing data cleaning to ensure accuracy. I thoroughly checked for misspellings and irregularities in string columns, enabling reliable analysis. Additionally, I calculated the ride duration in minutes and introduced new columns indicating the day of the week and month for each ride.

## Analysis:
I began by analyzing the bike type usage and found that electric bikes were the most popular choice for both casual and member riders, constituting approximately 22.04% and 29.09% of total rides, respectively.

Further analysis revealed that members took a significantly higher number of rides (59.31% of total rides) compared to casual riders (40.69% of total rides).

The average ride length for casual riders was longer (21.8 minutes) than for members (12.3 minutes), indicating that casual riders tend to use bikes for longer trips.

Analyzing ride length by day of the week and month revealed interesting patterns. For members, the average ride length was consistent throughout the week, while casual riders showed longer rides on weekends and during the summer months.

## Data Merging:
## Merging all datasets into one dataset using UNION ALL:
```sql
CREATE OR REPLACE TABLE divvy_tripdata. divvy_trip_datav1 AS 
(
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202202-divvy-tripdata` 
 UNION ALL
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202203-divvy-tripdata` 
 UNION ALL
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202204-divvy-tripdata` 
 UNION ALL
SELECT * 
 FROM `jaga-394318.divvy_tripdata.202205-divvy-tripdata_1` 
 UNION ALL
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202205-divvy-tripdata_2` 
 UNION ALL
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202206-divvy-tripdata_1` 
 UNION ALL
 SELECT * 
 FROM `jaga-394318.divvy_tripdata.202206-divvy-tripdata_2` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202207-divvy-tripdata_1` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202207-divvy-tripdata_2` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202208-divvy-tripdata_1` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202208-divvy-tripdata_2` 
 UNION ALL
   SELECT * 
  FROM `jaga-394318.divvy_tripdata.202209-divvy-tripdata_1` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202209-divvy-tripdata_2` 
 UNION ALL
   SELECT * 
  FROM `jaga-394318.divvy_tripdata.202210-divvy-tripdata_1` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202210-divvy-tripdata_2` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202211-divvy-tripdata` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202212-divvy-tripdata` 
 UNION ALL
  SELECT * 
 FROM `jaga-394318.divvy_tripdata.202301-divvy-tripdata` 
 )

```
## Query Result:
![tablev1](https://github.com/thejagadeesh/Cyclistic-bike-share-analysis/assets/114074976/cff76633-e055-4f6d-af37-07ba2a822b5f)

## Data Transformation and Cleanup: 
Creating a new table "divvy_trip_data_v2" with calculated columns for ride length, day of the week, and month:
```sql
CREATE TABLE divvy_tripdata.divvy_trip_datav2 AS
SELECT ride_id,
            rideable_type,
            started_at,
            ended_at,
            ROUND(TIMESTAMP_DIFF(ended_at, started_at, second)/60, 1) AS ride_length_minutes,
            EXTRACT(DAYOFWEEK FROM started_at) AS day_of_week,
            EXTRACT(MONTH FROM started_at) AS month,
            start_station_name,
            start_station_id,
            end_station_name,
            end_station_id,
            start_lat,
            start_lng,
            end_lat,
            end_lng,
            member_casual
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav1`;
```
## Query Result:
![image](https://github.com/thejagadeesh/Cyclistic-bike-share-analysis/assets/114074976/49ca4f83-d7bb-4919-a0ed-5d164bc1480a)

## Data Filtering:
Filter out erroneous data with negative ride lengths and rides lasting longer than a day:
```sql
CREATE TABLE divvy_tripdata.divvy_trip_datav3 AS
SELECT *
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav2`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440;
```
## Bike Type Usage and Percentages:
Calculate the count and percentages of rides for each member and casual rider by bike type:
```sql
SELECT
    member_casual,
    rideable_type,
    COUNT(1) AS total_rides,
    ROUND(100 * COUNT(1) / 5741891, 2) AS ride_percentage,
    COUNT(DISTINCT started_at) AS total_unique_dates
FROM
    `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY
    member_casual,
    rideable_type
ORDER BY
    member_casual;
```
Query Results:
| member_casual | rideable_type | total_rides | ride_percentage | total_unique_dates |
|---------------|---------------|-------------|-----------------|--------------------|
| casual        | electric_bike | 1,265,338   | 22.04%          | 1,207,357          |
| casual        | classic_bike  | 895,053     | 15.59%          | 854,268            |
| casual        | docked_bike   | 176,115     | 3.07%           | 172,941            |
| member        | electric_bike | 1,670,147   | 29.09%          | 1,592,473          |
| member        | classic_bike  | 1,735,238   | 30.22%          | 1,639,652          |
```
The query results reveal that electric bikes are the most popular choice for both casual and member riders, constituting approximately 22.04% and 29.09% of total rides, respectively, showcasing the significant adoption of electric bikes in the bike-sharing program.
```
## Count of Rides and Percentages by Member/Casual:
Calculate the count and percentages of rides for each member and casual rider:
```sql
SELECT
    member_casual,
    COUNT(*) AS count_rides,
    ROUND(COUNT(*) * 100 / SUM(COUNT(*)) OVER (), 2) AS percent
FROM
    `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY 
    member_casual
ORDER BY
    count_rides DESC;
```
Query Results:
| member_casual | count_rides | percent |
|---------------|-------------|---------|
| member        | 3,405,385   | 59.31%  |
| casual        | 2,336,506   | 40.69%  |

## Analyze Ride Length for All Riders:
Calculate the average, minimum, and maximum ride lengths for all riders:
```sql
SELECT AVG(ride_length_minutes) AS avg,
            MIN(ride_length_minutes) AS min,
            MAX(ride_length_minutes) AS max
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`;
```
Query Result:
| Row | Avg                | Min | Max     |
|-----|--------------------|-----|---------|
| 1   | 16.172750910806091 | 0.1 | 1439.9  |

## Round Trips Analysis:
Calculate the number of rides, the count of round trips, the percentage of round trips, and the count of unique dates for each member_casual category:
```sql
WITH RoundTrips AS (
  SELECT
    ride_id,
    started_at,
    ended_at,
    member_casual,
    ride_length_minutes,
    start_station_name,
    end_station_name,
    IF(start_station_name = end_station_name, 1, 0) AS is_round_trip
  FROM
    `jaga-394318.divvy_tripdata.divvy_trip_datav3`
)
SELECT
  member_casual,
  COUNT(1) AS count_rides,
  SUM(is_round_trip) AS count_round_trip,
  (ROUND(SAFE_DIVIDE(SUM(is_round_trip), COUNT(1)), 2) * 100) AS rate_round_percent,
  COUNT(DISTINCT DATE(started_at)) AS count_dates
FROM
  RoundTrips
GROUP BY
  member_casual;
```
Query Results:
| member_casual | count_rides | count_round_trip | rate_round_percent | count_dates |
|---------------|-------------|------------------|--------------------|-------------|
| casual        | 2,336,506   | 175,413          | 8.0%               | 365         |
| member        | 3,405,385   | 117,827          | 3.0%               | 365         |

## Average and Median Ride Lengths for Members and Casual Riders:
Calculate the average and median ride lengths for members and casual riders separately:
```sql
SELECT
  member_casual,
  ROUND(AVG(ride_length_minutes), 1) AS Avg_Ride_Length,
  ROUND(APPROX_QUANTILES(ride_length_minutes, 2)[OFFSET(1)], 1) AS median_Ride_Length
FROM
  `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY
  member_casual;

```
Query Result:
| member_casual | Avg_Ride_Length | median_Ride_Length |
|---------------|-----------------|--------------------|
| casual        | 21.8            | 12.8               |
| member        | 12.3            | 8.8                |

## Monthly and Day-of-the-Week Patterns:
Analyze the count of rides for each member and casual rider by month and day of the week:
## Average Ride Length for Members by Day of the Week
```sql
SELECT
    CASE day_of_week
        WHEN 1 THEN 'Sunday'
        WHEN 2 THEN 'Monday'
        WHEN 3 THEN 'Tuesday'
        WHEN 4 THEN 'Wednesday'
        WHEN 5 THEN 'Thursday'
        WHEN 6 THEN 'Friday'
        WHEN 7 THEN 'Saturday'
    END AS day_of_week_name,
    Round(AVG(ride_length_minutes), 2) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY day_of_week, day_of_week_name
ORDER BY day_of_week;
```
Query Result:
| Row | Day of Week | Avg Ride Length (Member) |
|-----|-------------|--------------------------|
| 1   | Sunday      | 13.62                    |
| 2   | Monday      | 11.91                    |
| 3   | Tuesday     | 11.71                    |
| 4   | Wednesday   | 11.76                    |
| 5   | Thursday    | 11.92                    |
| 6   | Friday      | 12.14                    |
| 7   | Saturday    | 13.75                    |

## Average Ride Length for Casuals by Day of the Week
```sql
SELECT
    CASE day_of_week
        WHEN 1 THEN 'Sunday'
        WHEN 2 THEN 'Monday'
        WHEN 3 THEN 'Tuesday'
        WHEN 4 THEN 'Wednesday'
        WHEN 5 THEN 'Thursday'
        WHEN 6 THEN 'Friday'
        WHEN 7 THEN 'Saturday'
    END AS day_of_week_name,
    Round(AVG(ride_length_minutes), 2) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY day_of_week, day_of_week_name
ORDER BY day_of_week;
```
Query Result:
| Row | Day of Week | Avg Ride Length (Member) |
|-----|-------------|--------------------------|
| 1   | Sunday      | 24.93                    | 
| 2   | Monday      | 22.20                    |
| 3   | Tuesday     | 19.45                    |
| 4   | Wednesday   | 18.76                    |
| 5   | Thursday    | 19.40                    |
| 6   | Friday      | 20.41                    |
| 7   | Saturday    | 24.49                    |

## Average Ride Length for Member Riders by Month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month_name,
    Round(AVG(ride_length_minutes), 2) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY month, month_name
ORDER BY month;
```
Query Result:
| Row | Month      | Avg Ride Length (Member) |
|-----|------------|--------------------------|
| 1   | January    | 10.07                    |
| 2   | February   | 11.06                    |
| 3   | March      | 11.71                    |
| 4   | April      | 11.36                    |
| 5   | May        | 13.06                    |
| 6   | June       | 13.66                    |
| 7   | July       | 13.44                    |
| 8   | August     | 13.09                    |
| 9   | September  | 12.63                    |
| 10  | October    | 11.54                    |
| 11  | November   | 10.86                    |
| 12  | December   | 10.34                    |

## Average Ride Length for Casual Riders by Month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month_name,
    Round(AVG(ride_length_minutes), 2) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY month, month_name
ORDER BY month;
```
Query Result:
| Row | Month      | Avg Ride Length         |
|-----|------------|-------------------------|
| 1   | January    | 13.71                   |
| 2   | February   | 19.63                   |
| 3   | March      | 24.24                   |
| 4   | April      | 23.28                   |
| 5   | May        | 25.56                   |
| 6   | June       | 23.42                   |
| 7   | July       | 23.19                   |
| 8   | August     | 21.49                   |
| 9   | September  | 20.06                   |
| 10  | October    | 18.48                   |
| 11  | November   | 15.55                   |
| 12  | December   | 13.40                   |

## Number of Rides Taken by Members in Each Month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month,
    COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY month
ORDER BY month;
```
Query Result:
| Row | Month      | Rides Taken |
|-----|------------|-------------|
| 1   | January    | 150082      |
| 2   | February   | 93883       |
| 3   | March      | 193825      |
| 4   | April      | 244212      |
| 5   | May        | 353864      |
| 6   | June       | 399510      |
| 7   | July       | 416827      |
| 8   | August     | 426427      |
| 9   | September  | 404139      |
| 10  | October    | 349209      |
| 11  | November   | 236671      |
| 12  | December   | 136736      |

##Number of Rides Taken by Casuals in Each Month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month,
    COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY month
ORDER BY month;
```
Query Result:
| Row | Month      | Rides Taken |
|-----|------------|-------------|
| 1   | July       | 404903      |
| 2   | June       | 367814      |
| 3   | August     | 357936      |
| 4   | September  | 295951      |
| 5   | May        | 279483      |
| 6   | October    | 208522      |
| 7   | April      | 125888      |
| 8   | November   | 100515      |
| 9   | March      | 89539       |
| 10  | December   | 44774       |
| 11  | January    | 39892       |
| 12  | February   | 21289       |

## Calculate the number of rides taken by Members on each day of the week
```sql
SELECT CASE
            WHEN day_of_week = 1 THEN 'Sunday'
            WHEN day_of_week = 2 THEN 'Monday'
            WHEN day_of_week = 3 THEN 'Tuesday'
            WHEN day_of_week = 4 THEN 'Wednesday'
            WHEN day_of_week = 5 THEN 'Thursday'
            WHEN day_of_week = 6 THEN 'Friday'
            WHEN day_of_week = 7 THEN 'Saturday' END AS day_of_the_week,
            COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY day_of_week
ORDER BY day_of_week;
```
Query Result:
| Row | Day of the Week | Rides Taken |
|-----|-----------------|-------------|
| 1   | Sunday          | 393537      |
| 2   | Monday          | 481848      |
| 3   | Tuesday         | 533481      |
| 4   | Wednesday       | 534985      |
| 5   | Thursday        | 540106      |
| 6   | Friday          | 475110      |
| 7   | Saturday        | 446318      |

## Calculate the number of rides taken by Casuals on each day of the week
```sql
SELECT CASE
            WHEN day_of_week = 1 THEN 'Sunday'
            WHEN day_of_week = 2 THEN 'Monday'
            WHEN day_of_week = 3 THEN 'Tuesday'
            WHEN day_of_week = 4 THEN 'Wednesday'
            WHEN day_of_week = 5 THEN 'Thursday'
            WHEN day_of_week = 6 THEN 'Friday'
            WHEN day_of_week = 7 THEN 'Saturday' END AS day_of_the_week,
            COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY day_of_week
ORDER BY day_of_week;
```
Query Result:
| Row | Day of the Week | Rides Taken |
|-----|-----------------|-------------|
| 1   | Sunday          | 391553      |
| 2   | Monday          | 280141      |
| 3   | Tuesday         | 267454      |
| 4   | Wednesday       | 277187      |
| 5   | Thursday        | 310961      |
| 6   | Friday          | 336223      |
| 7   | Saturday        | 472987      |

## Create a table to store the count of rides taken by members from each start station.
```sql
CREATE TABLE divvy_tripdata.start_station_count_members AS
SELECT start_station_name AS station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
ORDER BY COUNT(*) DESC;
```
- Create a table to store the count of rides taken by members to each end station.
```sql
CREATE TABLE divvy_tripdata.end_station_count_members AS
SELECT end_station_name AS station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
ORDER BY COUNT(*) DESC;
```
## Create a table to store the count of rides taken by casual riders from each start station.
```sql
CREATE TABLE divvy_tripdata.start_station_count_casual AS
SELECT start_station_name AS station_name,
            COUNT(*) AS number_of_casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
ORDER BY COUNT(*) DESC;
```
## Create a table to store the count of rides taken by casual riders to each end station.
```sql
CREATE TABLE divvy_tripdata.end_station_count_casual AS
SELECT end_station_name AS station_name,
            COUNT(*) AS number_of_casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
ORDER BY COUNT(*) DESC;
```
## count of rides taken by members for each month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month_name,
    member_casual,
    COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440 AND member_casual = 'member'
GROUP BY month, month_name, member_casual
ORDER BY month, member_casual;
```
Query Result:
| Row | Month     | Member Casual | Rides Taken |
|-----|-----------|---------------|-------------|
| 1   | January   | member        | 150082      |
| 2   | February  | member        | 93883       |
| 3   | March     | member        | 193825      |
| 4   | April     | member        | 244212      |
| 5   | May       | member        | 353864      |
| 6   | June      | member        | 399510      |
| 7   | July      | member        | 416827      |
| 8   | August    | member        | 426427      |
| 9   | September | member        | 404139      |
| 10  | October   | member        | 349209      |
| 11  | November  | member        | 236671      |
| 12  | December  | member        | 136736      |

## Count of rides taken by casuals for each month
```sql
SELECT
    CASE month
        WHEN 1 THEN 'January'
        WHEN 2 THEN 'February'
        WHEN 3 THEN 'March'
        WHEN 4 THEN 'April'
        WHEN 5 THEN 'May'
        WHEN 6 THEN 'June'
        WHEN 7 THEN 'July'
        WHEN 8 THEN 'August'
        WHEN 9 THEN 'September'
        WHEN 10 THEN 'October'
        WHEN 11 THEN 'November'
        WHEN 12 THEN 'December'
    END AS month_name,
    member_casual,
    COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440 AND member_casual = 'casual'
GROUP BY month, month_name, member_casual
ORDER BY month, member_casual;
```
Query Result:
| Row | Month     | Member Casual | Rides Taken |
|-----|-----------|---------------|-------------|
| 1   | January   | casual        | 39892       |
| 2   | February  | casual        | 21289       |
| 3   | March     | casual        | 89539       |
| 4   | April     | casual        | 125888      |
| 5   | May       | casual        | 279483      |
| 6   | June      | casual        | 367814      |
| 7   | July      | casual        | 404903      |
| 8   | August    | casual        | 357936      |
| 9   | September | casual        | 295951      |
| 10  | October   | casual        | 208522      |
| 11  | November  | casual        | 100515      |
| 12  | December  | casual        | 44774       |

## Calculate the number of rides taken by members and casual riders
```sql
SELECT member_casual,
            COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY member_casual;
```
Query Result:
| Row | Member Casual | Rides Taken |
|-----|---------------|-------------|
| 1   | casual        | 2336506     |
| 2   | member        | 3405385     |

## Member Rides Count by Rideable Type
```sql
SELECT rideable_type,
            COUNT(*) AS member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0
            AND ride_length_minutes < 1440
            AND member_casual = 'member'
GROUP BY rideable_type
```
Query Result:
| Row | Rideable Type | Member Rides |
|-----|---------------|--------------|
| 1   | electric_bike | 1670147      |
| 2   | classic_bike  | 1735238      |

## Casual Rides Count by Rideable Type
```sql
SELECT rideable_type,
            COUNT(*) AS casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0
            AND ride_length_minutes < 1440
            AND member_casual = 'casual'
GROUP BY rideable_type
```
Query Result:
| Row | Rideable Type | Casual Rides |
|-----|---------------|--------------|
| 1   | electric_bike | 1265338      |
| 2   | docked_bike   | 176115       |
| 3   | classic_bike  | 895053       |

## Calculate the total number of visits to start station for members and casual riders
```sql
SELECT start_station_name AS station_name,
       member_casual,
       COUNT(*) AS total_visits
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440
GROUP BY start_station_name, member_casual;
```
Query Result:
| Row | Station Name                             | Member/Casual | Total Visits |
|-----|------------------------------------------|---------------|--------------|
| 1   | Monticello Ave & Chicago Ave             | member        | 92           |
| 2   | Lamon Ave & Chicago Ave                  | casual        | 132          |
| 3   | Lamon Ave & Chicago Ave                  | member        | 56           |
| 4   | Kildare Ave & Division St                | casual        | 38           |
| 5   | Public Rack - Kildare Ave & Division St  | casual        | 48           |
| 6   | Public Rack - Kildare Ave & Division St  | member        | 29           |
| 7   | Lavergne Ave & Division St               | casual        | 61           |
| 8   | Lavergne Ave & Division St               | member        | 22           |
| 9   | Menard Ave & Division St                 | casual        | 59           |
| 10	 | Leamington Ave & Hirsch St               | casual        | 83           |

## Finding the most popular start stations for member riders
```sql
SELECT start_station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes < 1440
            AND ride_length_minutes > 0
            AND member_casual = 'member'
            AND start_station_name != 'null'
GROUP BY start_station_name
ORDER BY COUNT(*) DESC
```
Query Result:
| Row |    Start Station Name       | Number of Member Rides |
|-----|-----------------------------|------------------------|
| 1   | Kingsbury St & Kinzie St    | 25,208                 |
| 2   | Clark St & Elm St           | 22,475                 |
| 3   | Wells St & Concord Ln       | 21,555                 |
| 4   | University Ave & 57th St    | 21,087                 |
| 5   | Clinton St & Washington Blvd| 20,611                 |
| 6   | Ellis Ave & 60th St         | 20,521                 |
| 7   | Loomis St & Lexington St    | 19,514                 |
| 8   | Wells St & Elm St           | 19,379                 |
| 9   | Clinton St & Madison St     | 19,232                 |
| 10  | Broadway & Barry Ave        | 18,004                 |

## Finding the most popular start stations for casual riders
```sql
SELECT start_station_name,
            COUNT(*) AS number_of_casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes < 1440
            AND ride_length_minutes > 0
            AND member_casual = 'casual'
            AND start_station_name != 'null'
GROUP BY start_station_name
ORDER BY COUNT(*) DESC
```
Query Result:
| Row |    Start Station Name              | Number of Casual Rides |
|-----|------------------------------------|------------------------|
| 1   | Streeter Dr & Grand Ave            | 58,091                 |
| 2   | DuSable Lake Shore Dr & Monroe St  | 31,869                 |
| 3   | Millennium Park                    | 25,553                 |
| 4   | Michigan Ave & Oak St              | 25,276                 |
| 5   | DuSable Lake Shore Dr & North Blvd | 23,631                 |
| 6   | Shedd Aquarium                     | 20,412                 |
| 7   | Theater on the Lake                | 18,431                 |
| 8   | Wells St & Concord Ln              | 16,304                 |
| 9   | Dusable Harbor                     | 14,083                 |
| 10  | Clark St & Armitage Ave            | 13,840                 |


## Count Number of Member Rides per End Station
```sql
SELECT end_station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes < 1440
            AND ride_length_minutes > 0
            AND member_casual = 'member'
            AND end_station_name != 'null'
GROUP BY end_station_name
ORDER BY COUNT(*) DESC
```
Query Result:
| Row |        End Station Name       | Number of Member Rides |
|-----|-------------------------------|-----------------------|
| 1   | Kingsbury St & Kinzie St      | 25,073                |
| 2   | Clark St & Elm St             | 22,808                |
| 3   | Wells St & Concord Ln         | 22,201                |
| 4   | University Ave & 57th St      | 21,609                |
| 5   | Clinton St & Washington Blvd  | 21,389                |
| 6   | Ellis Ave & 60th St           | 20,345                |
| 7   | Clinton St & Madison St       | 20,071                |
| 8   | Loomis St & Lexington St      | 19,358                |
| 9   | Wells St & Elm St             | 19,094                |
| 10  | Broadway & Barry Ave          | 18,353                |

## Count Number of Casual Rides per End Station
```sql
SELECT end_station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes < 1440
            AND ride_length_minutes > 0
            AND member_casual = 'casual'
            AND end_station_name != 'null'
GROUP BY end_station_name
ORDER BY COUNT(*) DESC
```
Query Result:
| Rank |         End Station Name           | Number of Member Rides |
|------|------------------------------------|------------------------|
| 1    | Streeter Dr & Grand Ave            | 60,047                |
| 2    | DuSable Lake Shore Dr & Monroe St  | 29,655                |
| 3    | Millennium Park                    | 26,841                |
| 4    | Michigan Ave & Oak St              | 26,505                |
| 5    | DuSable Lake Shore Dr & North Blvd | 26,174                |
| 6    | Theater on the Lake                | 19,435                |
| 7    | Shedd Aquarium                     | 18,779                |
| 8    | Wells St & Concord Ln              | 15,631                |
| 9    | Clark St & Armitage Ave            | 13,887                |
| 10   | Clark St & Lincoln Ave             | 13,626                |


## Calculate the total number of visits to each station for members by joining the start_station_count_members and end_station_count_members tables.

-- We are using a JOIN operation on the station_name column to match the stations between the two tables.

-- Then, we are adding the number_of_member_rides from both tables to get the total_member_visits for each station.
```sql
SELECT
    start_count.station_name,
    start_count.number_of_member_rides AS total_member_visits
FROM
    jaga-394318.divvy_tripdata.start_station_count_members start_count
JOIN
    jaga-394318.divvy_tripdata.end_station_count_members end_count
ON
    start_count.station_name = end_count.station_name
ORDER BY
    total_member_visits DESC;
```
Query Result:
| Rank |             Station Name            | Total Member Visits |
|------|-------------------------------------|--------------------|
| 1    | Streeter Dr & Grand Ave             | 75,303             |
| 2    | DuSable Lake Shore Dr & Monroe St   | 41,276             |
| 3    | DuSable Lake Shore Dr & North Blvd  | 40,099             |
| 4    | Michigan Ave & Oak St               | 39,762             |
| 5    | Wells St & Concord Ln               | 37,859             |
| 6    | Clark St & Elm St                   | 35,554             |
| 7    | Millennium Park                     | 35,065             |
| 8    | Kingsbury St & Kinzie St            | 34,101             |
| 9    | Theater on the Lake                 | 32,981             |
| 10   | Wells St & Elm St                   | 31,910             |

## calculate the total number of casual visits to each station by adding the number_of_casual_rides from the start_station_count_casual table.

-- with the number_of_casual_rides from the end_station_count_casual table. The tables are joined on the station_name column.

-- Selecting the station_name column from the start_station_count_casual table as station_name.

```sql
SELECT 
    start_count.station_name AS station_name,
    -- Adding the number_of_casual_rides from both start_count and end_count tables to get total_casual_visits
    start_count.number_of_casual_rides + end_count.number_of_casual_rides AS total_casual_visits
-- Using the alias 'start_count' for the start_station_count_casual table
FROM 
    jaga-394318.divvy_tripdata.start_station_count_casual start_count
-- Using the alias 'end_count' for the end_station_count_casual table
JOIN 
    jaga-394318.divvy_tripdata.end_station_count_casual end_count
-- Joining the tables on the matching station_name column
ON 
    start_count.station_name = end_count.station_name
-- Ordering the results in descending order of total_casual_visits
ORDER BY 
    total_casual_visits DESC;
```
Query Result:

| Rank | Station Name                          |Total Casual Visits |
| ---- | ------------------------------------  | ------------------ |
| 1    | Streeter Dr & Grand Ave               | 150,895            |
| 2    | DuSable Lake Shore Dr & North Blvd    | 82,326             |
| 3    | DuSable Lake Shore Dr & Monroe St     | 81,441             |
| 4    | Michigan Ave & Oak St                 | 80,065             |
| 5    | Wells St & Concord Ln                 | 75,691             |
| 6    | Clark St & Elm St                     | 70,579             |
| 7    | Millennium Park                       | 70,520             |
| 8    | Kingsbury St & Kinzie St              | 67,008             |
| 9    | Theater on the Lake                   | 66,051             |
| 10   | Wells St & Elm St                     | 62,712             |

# Dashboard
![Cylistic Dashboard](https://github.com/thejagadeesh/Cyclistic-bike-share-analysis/assets/114074976/91de2f15-7270-4767-aa34-ce0e1aadef4a)

## Tableau Dashboard Link:
https://public.tableau.com/views/CyclisticBike-ShareAnalysis_16909122669020/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link

## Key Findings:
* Electric bikes are the most popular choice for both casual and member riders, constituting approximately 22.04% and 29.09% of total rides, respectively.
* Members take a significantly higher number of rides (59.31% of total rides) compared to casual riders (40.69% of total rides).
* Casual riders tend to use bikes for longer trips, with an average ride length of 21.8 minutes, compared to 12.3 minutes for members.
* There is a significant percentage of round trips taken by casual riders (8.0%) compared to member riders (3.0%).
* Casual riders take longer rides on weekends and during the summer months, while members show consistent ride lengths throughout the week.
* The top start stations for member riders are Kingsbury St & Kinzie St, Clark St & Elm St, and Wells St & Concord Ln, while the top start station for casual riders is Streeter Dr & Grand Ave.
* The top end stations for member riders are Kingsbury St & Kinzie St, Clark St & Elm St, and Wells St & Concord Ln, while the top end station for casual riders is Streeter Dr & Grand Ave.

## Insights:
* Electric bikes have gained widespread popularity among both casual and member riders, indicating that offering more electric bikes could attract even more customers.
* Members are more engaged and frequent users of the bike-share service, making them an ideal target for membership retention and loyalty programs.
* Casual riders might be attracted to the service due to longer trip durations. Offering special deals or promotions for longer rides could encourage casual riders to become annual members.
* The higher percentage of round trips taken by casual riders suggests that they might be using the bikes for leisure activities or short commutes. Tailoring marketing strategies to highlight the convenience of round trips might attract more casual riders.

## Recommendations:
## Promote Electric Bikes: 
Launch marketing campaigns focusing on the convenience and benefits of electric bikes to encourage both casual and member riders to choose electric bikes more frequently.
## Targeted Marketing for Casual Riders: 
Develop targeted marketing campaigns aimed at converting casual riders into annual members. Offer discounts, loyalty programs, or bundle deals to incentivize them to sign up for annual memberships.
## Promote Longer Rides: 
Create promotions or incentives that reward users for taking longer rides. For example, offer discounted rates for rides exceeding a certain duration.
## Focus on Round Trips: 
Design marketing materials highlighting the convenience and flexibility of round trips. Encourage casual riders to use the bikes for short errands or sightseeing, emphasizing the ease of returning to their starting point.
## Conclusion:
The analysis revealed significant differences in usage patterns between casual and member riders. Electric bikes are popular among both groups, but members take more rides and have shorter average ride lengths. Casual riders, on the other hand, tend to take longer trips and have a higher percentage of round trips. To boost annual memberships, targeted marketing strategies should be designed to cater to the specific preferences of casual riders, promote the benefits of electric bikes, and incentivize longer rides. By implementing these recommendations, Cyclistic can drive growth in the number of annual members and increase overall customer engagement with the bike-share service.
