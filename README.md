# Cyclistic-bike-share-analysis

```sql

SELECT *
 FROM
   `jaga-394318.divvy_tripdata.`.INFORMATION_SCHEMA.COLUMN_FIELD_PATHS
 WHERE
   table_name = "202202-divvy-tripdata"    OR
   table_name = "202203-divvy-tripdata"    OR
   table_name = "202204-divvy-tripdata"    OR
   table_name = "202205-divvy-tripdata_1"  OR
   table_name = "202205-divvy-tripdata_2"  OR
   table_name = "202206-divvy-tripdata_1"  OR
   table_name = "202206-divvy-tripdata_2"  OR
   table_name = "202207-divvy-tripdata_1"  OR
   table_name = "202207-divvy-tripdata_2"  OR
   table_name = "202208-divvy-tripdata_1"  OR
   table_name = "202208-divvy-tripdata_2"  OR
   table_name = "202209-divvy-tripdata_1"  OR
   table_name = "202209-divvy-tripdata_2"  OR
   table_name = "202210-divvy-tripdata_1"  OR
   table_name = "202210-divvy-tripdata_2"  OR
   table_name = "202211-divvy-tripdata"    OR
   table_name = "202212-divvy-tripdata"    OR
   table_name = "202301-divvy-tripdata"  
 ORDER BY column_name, table_name;
```

#Need to merge all of the datasets into one dataset using UNION ALL
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
-- Create a new table "divvy_trip_datav2" with calculated columns for ride length, day of the week, and month
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

-- Filter out erroneous data with negative ride lengths and rides lasting longer than a day
```sql
CREATE TABLE divvy_tripdata.divvy_trip_datav3 AS
SELECT *
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav2`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440;

-- Analyze ride length for all riders
SELECT AVG(ride_length_minutes) AS avg,
            MIN(ride_length_minutes) AS min,
            MAX(ride_length_minutes) AS max
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`;
```
```sql
Row	avg	               min	max	
1	16.172750910806091	0.1	1439.9
```

-- Calculate average ride length for members and casual riders
```sql
SELECT member_casual,
            AVG(ride_length_minutes) AS avg_ride_length
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY member_casual;
```
```sql
Row	member_casual	avg_ride_length	
1	casual	21.771984707079753	
2	member	12.3309992849560
```
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
    AVG(ride_length_minutes) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY day_of_week, day_of_week_name
ORDER BY day_of_week;
```
```sql
Row	day_of_week_name	 avg_ride_length_member	
1	Sunday	             13.627357021067917	
2	Monday	             11.91083204662049	
3	Tuesday	11.711094865609073	
4	Wednesday	11.767330485901484	
5	Thursday	11.92166130352191	
6	Friday	12.145869377617808	
7	Saturday	13.75060719038890	
```
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
    AVG(ride_length_minutes) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY day_of_week, day_of_week_name
ORDER BY day_of_week;
```
```sql
Row	day_of_week_name	avg_ride_length_member	
1	Sunday	24.935734370570547	
2	Monday	22.205314466643586	
3	Tuesday	19.450710402536494	
4	Wednesday	18.767935364934146	
5	Thursday	19.40566405433476	
6	Friday	20.410732757723341	
7	Saturday	24.49270656487392	
```
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
    AVG(ride_length_minutes) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'member'
GROUP BY month, month_name
ORDER BY month;
```
```sql
Row	month_name	avg_ride_length_member	
Load more
6	June	13.663008435333284	
7	July	13.441689477888913	
8	August	13.091214674492942	
9	September	12.637538322210908	
10	October	11.54782980965555	
11	November	10.868295228397228	
12	December	10.349790837818858	
```
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
    AVG(ride_length_minutes) AS avg_ride_length_member
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE member_casual = 'casual'
GROUP BY month, month_name
ORDER BY month;
```
```sql
Row	month_name	avg_ride_length_member	
Load more
6	June	23.427427177867113	
7	July	23.199642877429856	
8	August	21.495009722408454	
9	September	20.060855344296861	
10	October	18.483160529824186	
11	November	15.555658359448838	
12	December	13.4084558002412	
```
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
```sql
Row	month	rides_taken	
Load more
6	July	416827	
7	June	399510	
8	March	193825	
9	May	353864	
10	November	236671	
11	October	349209	
12	September	404139	
```
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
```sql
Row	month	rides_taken	
Load more
6	October	208522	
7	April	125888	
8	November	100515	
9	March	89539	
10	December	44774	
11	January	39892	
12	February	21289	
```
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
```sql
Row	month	rides_taken	
1	July	404903	
2	June	367814	
3	August	357936	
4	September	295951	
5	May	279483	
6	October	208522	
7	April	125888	
8	November	100515	
9	March	89539	
10	December	44774	
11	January	39892	
12	February	21289	
```
-- Calculate the number of rides taken by each group on each day of the week
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
```sql
Row	day_of_the_week	rides_taken	
1	Sunday	393537	
2	Monday	481848	
3	Tuesday	533481	
4	Wednesday	534985	
5	Thursday	540106	
6	Friday	475110	
7	Saturday	446318	
```

-- Calculate the number of rides taken by each group on each day of the week
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
```sql
Row	day_of_the_week	rides_taken	
1	Sunday	391553	
2	Monday	280141	
3	Tuesday	267454	
4	Wednesday	277187	
5	Thursday	310961	
6	Friday	336223	
7	Saturday	472987	
```

-- Calculate the total number of visits to each station for members and casual riders
```sql
CREATE TABLE divvy_tripdata.start_station_count_members AS
SELECT start_station_name AS station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
ORDER BY COUNT(*) DESC;
```
```sql
CREATE TABLE divvy_tripdata.end_station_count_members AS
SELECT end_station_name AS station_name,
            COUNT(*) AS number_of_member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
ORDER BY COUNT(*) DESC;
```
```sql
CREATE TABLE divvy_tripdata.start_station_count_casual AS
SELECT start_station_name AS station_name,
            COUNT(*) AS number_of_casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE start_station_name IS NOT NULL
GROUP BY start_station_name
ORDER BY COUNT(*) DESC;
```
```sql
CREATE TABLE divvy_tripdata.end_station_count_casual AS
SELECT end_station_name AS station_name,
            COUNT(*) AS number_of_casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE end_station_name IS NOT NULL
GROUP BY end_station_name
ORDER BY COUNT(*) DESC;
```

-- Calculate the number of rides taken by each group in each month
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
```sql
Row	month_name	member_casual	rides_taken	
1	January	member	150082	
2	February	member	93883	
3	March	member	193825	
4	April	member	244212	
5	May	member	353864	
6	June	member	399510	
7	July	member	416827	
8	August	member	426427	
9	September	member	404139	
10	October	member	349209	
11	November	member	236671	
12	December	member	136736	
```
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
```sql
Row	month_name	member_casual	rides_taken	
1	January	casual	39892	
2	February	casual	21289	
3	March	casual	89539	
4	April	casual	125888	
5	May	casual	279483	
6	June	casual	367814	
7	July	casual	404903	
8	August	casual	357936	
9	September	casual	295951	
10	October	casual	208522	
11	November	casual	100515	
12	December	casual	44774	
```
-- Calculate the number of rides taken by members and casual riders
```sql
SELECT member_casual,
            COUNT(*) AS rides_taken
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
GROUP BY member_casual;
```
```sql
Row	member_casual	rides_taken	
1	casual	2336506	
2	member	3405385
```
```sql
SELECT rideable_type,
            COUNT(*) AS member_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0
            AND ride_length_minutes < 1440
            AND member_casual = 'member'
GROUP BY rideable_type
```
```sql
Row	rideable_type	member_rides	
1	electric_bike	1670147	
2	classic_bike	1735238
```
```sql
SELECT rideable_type,
            COUNT(*) AS casual_rides
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0
            AND ride_length_minutes < 1440
            AND member_casual = 'casual'
GROUP BY rideable_type
```
```sql
Row	rideable_type	casual_rides	
1	electric_bike	1265338	
2	docked_bike	176115	
3	classic_bike	895053
```

-- Calculate the total number of visits to each station for members and casual riders
```sql
SELECT start_station_name AS station_name,
       member_casual,
       COUNT(*) AS total_visits
FROM `jaga-394318.divvy_tripdata.divvy_trip_datav3`
WHERE ride_length_minutes > 0 AND ride_length_minutes < 1440
GROUP BY start_station_name, member_casual;
```
```sql
Row	station_name	member_casual	total_visits	
Load more
8	Public Rack - Kildare Ave & Division St	casual	48	
9	Public Rack - Kildare Ave & Division St	member	29	
10	Lavergne Ave & Division St	casual	61	
11	Lavergne Ave & Division St	member	22	
12	Menard Ave & Division St	casual	59	
13	Leamington Ave & Hirsch St	casual	83	
14	Major Ave & Bloomingdale Ave	casual	82	
15	Major Ave & Bloomingdale Ave	member	25	
16	Narragansett & McLean	casual	80	
17	Narragansett & McLean	member	99	
```
--Finding most popular start stations for members and casual riders…
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
```sql

Row	start_station_name	number_of_member_rides	
1	Kingsbury St & Kinzie St	25208	
2	Clark St & Elm St	22475	
3	Wells St & Concord Ln	21555	
4	University Ave & 57th St	21087	
5	Clinton St & Washington Blvd	20611	
6	Ellis Ave & 60th St	20521	
7	Loomis St & Lexington St	19514	
8	Wells St & Elm St	19379	
9	Clinton St & Madison St	19232	
10	Broadway & Barry Ave	18004	
```
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
```sql
Row	start_station_name	number_of_casual_rides	
Load more
2	DuSable Lake Shore Dr & Monroe St	31869	
3	Millennium Park	25553	
4	Michigan Ave & Oak St	25276	
5	DuSable Lake Shore Dr & North Blvd	23631	
6	Shedd Aquarium	20412	
7	Theater on the Lake	18431	
8	Wells St & Concord Ln	16304	
9	Dusable Harbor	14083	
10	Clark St & Armitage Ave	13840	

```

--Finding the most popular end stations for members and casual riders…
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
```sql
Row	end_station_name	number_of_member_rides	
1	Kingsbury St & Kinzie St	25073	
2	Clark St & Elm St	22808	
3	Wells St & Concord Ln	22201	
4	University Ave & 57th St	21609	
5	Clinton St & Washington Blvd	21389	
6	Ellis Ave & 60th St	20345	
7	Clinton St & Madison St	20071	
8	Loomis St & Lexington St	19358	
9	Wells St & Elm St	19094	
10	Broadway & Barry Ave	18353	
```

--Finding the most popular end stations for members and casual riders…
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
```sql
Row	end_station_name	number_of_member_rides	
Load more
4	Michigan Ave & Oak St	26505	
5	DuSable Lake Shore Dr & North Blvd	26174	
6	Theater on the Lake	19435	
7	Shedd Aquarium	18779	
8	Wells St & Concord Ln	15631	
9	Clark St & Armitage Ave	13887	
10	Clark St & Lincoln Ave	13626	
```

