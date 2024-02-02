# Cyclistic-Case-Study

### Project Overview 
---

This Data Analysis projects aims to provide insights into usage of the cyclistic program by the members and casual users. By Analyzing various aspects of the cycle trips data we can identify trends, make Data-driven recommendation to improve the company. 


### Scenario
---

In the given scenario, I am a data analyst working in the marketing analyst team at Cyclistic,a bike share company in Chicago.
The director of marketing believes the company’s future success depends on maximizing the number of annual memberships.
Therefore, my team wants to understand how casual riders and annual members use Cyclistic bikes differently.

### Key Stakholders

#### Primary Stakeholders:
Lily Moreno - Director of Marketing,
Cyclistic Executive Team
#### Secondary Stakeholders:
Marketing Analytics Team 

### Data Analysis
---

- #### ASK
- #### PREPARE
- #### PROCESS
- #### ANALYZE
- #### SHARE
- #### ACT


### Ask
---

#### Identify the business task

The key business task in this case is to discover how casual riders and Cyclistic members use their rental bikes differently. Both the Director of Marketing as well as finance analysts have concluded that annual members are more profitable.

Therefore, the results of this analysis will be used to design a new marketing strategy to convert casual riders to annual members.

1. How do members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?
   

### Preparing Data
---


### Data Sources Used

The datasets used in this analysis have been made available by Motivate Internation Inc. 

This is public data that you can use to explore how different customer types are using Cyclistic bikes. 

Note: Cyclistic is a fictional company, hence the datasets contain the name of a different company. For the purpose of this analysis, the given datasets can be used to answer the business task 


### Tools Used

- SQL - Postgresql - Data Cleaning and Data Analysis.
- Tableau - Data Visualization.


### Creating tables and importing the datasets

- I am using 6 months of data (from 2020-04 to 2020-09) 

```sql
create table cycle_trip_04(
ride_id varchar,
rideable_type varchar,
started_at timestamp,
ended_at timestamp,
start_station_name varchar,
start_station_id integer,
end_station_name varchar,
end_station_id integer,
start_lat numeric,
start_lng numeric,
end_lat numeric,
end_lng numeric,
member_casual varchar);

copy cycle_trip_04 from 'F:\Data Analytics\Case study\Cyclistic\202004-divvy-tripdata.csv' csv header;
```
uploaded all the other dataset by the above method

### Processing data (Data Cleaning)
---

- Check for null values
- Check for the formats
- Check for Duplicates
- Check for White spaces
- Check for incomplete and wrong data(spelling mistakes,irrelevant data etc.)

### Cleaning
- I can find out the time it takes for every ride to complete by subtracting started_at from ended_at

```sql

select *, (ended_at - started_at) as ride_length
from cycle_trip_04

```
- By doing this I found that there are some negative values in ride_length in all the datasets,
-  i.e. ended_at time was less than started_at ex- started_at - '2020-04-02 18:30:00' , ended_at - '2020-04-02 18:25:00' this is not possible so I gathered that the values were entered in wrong columns so I swapped the values of ended_at and started_at where the difference was in negative.

```sql

update cycle_trip_04
set started_at = ended_at,
	ended_at = started_at	
where (ended_at-started_at) < '00:00:00'

```
and did that for all the other datasets.

By looking at the datasets I know that all 6 datasets that I am going to work with have matching column names and those columns have same data types, so I combine them into one using union all

```sql

create view cycletripsall as
 
(select * from cycle_trip_04
union all
select * from cycle_trip_05
union all
select * from cycle_trip_06
union all
select * from cycle_trip_07
union all
select * from cycle_trip_08
union all
select * from cycle_trip_09)

```
Then I checked for null values,incomplete data and spelling mistakes in all the columns.

```sql

select * from cycletripsall
where started_at is null

select * from cycletripsall
where ended_at is null

select * from cycletripsall
where ride_id is null

```
and so on.


- We can find out the number of characters are there for every ride_id


```sql

select length(ride_id) from cycletripsall

```

There are 16 characters.
By knowing this we can check if there are ids which does not have same no. of characters.


```sql

select ride_id from cycletripsall
where length(ride_id) != '16'

```

There were none.


By knowing the days I can know total trips for every day of the week.


```sql

select *, (ended_at-started_at) as ride_length, to_char(started_at, 'DY') as day_started_at
from cycletripsall

```

Now i saved this as a view 


```sql

create view cycletrips as
select *, (ended_at-started_at) as ride_length, to_char(started_at, 'DY') as day_started_at
from cycletripsall

```

Now the data is ready for analysis.


### Analyze
---

1. Finding total trips of casual and member users and their minimum ride_length, maximum ride_length and average ride_length.
  

```sql

select
member_casual,
count(member_casual) as total,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length,
avg(ride_length) as avg_ride_length

from cycletrips

group by member_casual

```

#### Output

![Screenshot (96)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/781d56c9-68d7-497a-87c7-053c7b70f522)



2. Finding number of rides for every day of the week for casual and member users
  

```sql

select member_casual,
started_at_day,
count(ride_id) as rides

from cycletrips

group by member_casual,started_at_day
order by member_casual,rides desc

```

#### Output

![Screenshot (58)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/ecab46cd-8bdb-4108-92e5-33685d708668)


3. Finding the total number of casual and member users who ride for the following time intervals:

- 1 min to 10 mins
- 10.01 mins to 20 mins
- 20.01 mins to 30 mins
- 30.01 mins to 40 mins
- 40.01 mins to 50 mins
- 50.01 mins to 1 hour
  and so on


```sql

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:01:00' and '00:10:00'  
group by member_casual
 
select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:10:01' and '00:20:00'  
group by member_casual
 
select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:20:01' and '00:30:00'  
group by member_casual
 
select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:30:01' and '00:40:00'  
group by member_casual
 
select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:40:01' and '00:50:00'  
group by member_casual
 
select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:50:01' and '01:00:00'  
group by member_casual

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '01:00:01' and '02:00:00' 
group by member_casual

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '02:00:01' and '03:00:00' 
group by member_casual

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '03:00:01' and '04:00:00' 
group by member_casual

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '04:00:01' and '23:59:59' 
group by member_casual

select member_casual , 
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length >= '1 day'
group by member_casual

```

#### Output


![Screenshot (66)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/c81f3788-072b-4a75-bda3-9ba8db664be3) 
![Screenshot (67)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/60cbbbf8-5082-4f97-aa1a-893f19b49e09)

![Screenshot (68)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/5fc4ee8e-4489-412e-83ff-3dc82fad2335) 
![Screenshot (69)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/17e26531-35cc-4df1-bdca-5904dad8d5d5)

![Screenshot (70)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/9f085633-aeb9-4ef6-9492-6203d70ebafa) 
![Screenshot (71)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/e7c3b6ae-e07d-4b89-9f84-d2bea9f156dc)

![Screenshot (72)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/bf4f512f-88e0-4569-ba6e-330568a3292e)
![Screenshot (75)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/eca2d09e-6d60-4d7a-aaa7-dc278fe1af79)
![Screenshot (76)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/34345838-c46b-43bf-b1e1-f8d23c815821)

![Screenshot (81)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/1c0a99a2-fe3a-438c-ba13-49f49c51a398)
![Screenshot (79)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/06c29fe0-8ec2-4cb3-a4ae-f5ac38136d03)


- By Analyzing the data we found out that the most users ride between these durations:
  - 1 min to 10 mins
  - 10 mins to 20 mins
  - 20 mins to 30 mins
  - 30 mins to 40 mins
  - 40 mins to 50 mins
  - 1 hour to 2 hour

4. Now analyzing these duration for every day of the week.


```sql

select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:01:00' and '00:10:00'  
group by member_casual,started_at_day
order by member_casual, total_count desc
 
select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:10:01' and '00:20:00'  
group by member_casual,started_at_day
order by member_casual, total_count desc
 
select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:20:01' and '00:30:00'  
group by member_casual,started_at_day
order by member_casual, total_count desc
 
select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:30:01' and '00:40:00'  
group by member_casual,started_at_day
order by member_casual, total_count desc
 
select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '00:40:01' and '00:50:00'  
group by member_casual,started_at_day 
order by member_casual, total_count desc
 
select member_casual , started_at_day,
count(ride_id) as total_count,
min(ride_length) as min_ride_length,
max(ride_length) as max_ride_length
from cycletrips
where ride_length between '01:00:01' and '02:00:00'  
group by member_casual,started_at_day  
order by member_casual, total_count desc


```

### Output 


![Screenshot (89)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/af942048-a4f0-4f49-ab73-5a619f831779)
![Screenshot (90)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/505ec36b-0ccd-4685-86ba-999fe798be47)
![Screenshot (91)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/b352f1a1-4959-434a-b573-c308b779b923)
![Screenshot (92)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/696cb461-2cb7-42d4-b7fc-6e022e1ceb21)
![Screenshot (93)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/0fcdd934-8885-47f0-9e62-11e3f349257b)
![Screenshot (94)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/61244692-0536-47c7-81b3-d5d8971ae796)

- The most number of trips were from these following days Saturday, Sunday and Friday.

5. Now finding out the top 10 used routes for the users.

```sql

- For casual users 

select 
start_station_name,
end_station_name,
count(ride_id) as rides

from cycletrips
where member_casual = 'casual' and start_station_name is not null and end_station_name is not null
group by start_station_name,end_station_name
order by rides desc

limit 10

- For members

select 
start_station_name,
end_station_name,
count(ride_id) as rides

from cycletrips
where member_casual = 'casual' and start_station_name is not null and end_station_name is not null
group by start_station_name,end_station_name
order by rides desc

limit 10


```

### Output

Casual :

![Screenshot (99)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/00ed4426-281e-49aa-8503-637cbce9b2e5)

Member :

![Screenshot (100)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/452f45c4-da92-41c1-a300-d9c4a40ef23c)


### Share

- Now visualizing the Data we found by Analyzing.

- Number of rides for every day of the week for casual and member users

  
![DOWALL](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/c870137b-3bff-4018-8a24-5760bc752fe7)


- Number of rides for ride duration
  

![RLCMCDASH](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/9b3d3200-6742-4474-8b0e-eb138ee20b82)



- Number of rides for each month

![TRIPSMONTH](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/0737ae94-e198-4957-a0a3-dec87b6b9d61)


## Summary of Analysis 

This analysis is done on the data of these 6 months (2020/04 - 2020/09)


- Casual users ride for  49 minutes on average.
- Members ride for 17 minutes on average.

- The most rides are in between these ride lengths 1 min - 10 mins, 10 mins - 20 mins, 20- 30 mins for members.
- The most rides are in between these ride lengths 1 min - 10 mins, 10 mins - 20 mins, 20- 30 mins and 1 hour - 2 hour for Casual users.
  
- The most number of rides were on Saturday for both the users.
- The trips are low on Monday and increases at Friday reaches peak on Saturday with slight drop off on Sunday.

- Members use the bikes more consistently throughout the week.
- Casual use the bike more on Saturday, Sunday and Friday.
- Casual users ride atleast 2 times longer on any day of the week than members.

- Most of the top 10 routes of the Casual and Member users are not same.

- The most number of rides for both the users occur in these months June, August and September were August has most number of rides.

  # Act

  ---

How do members and casual riders use Cyclistic bikes differently?

- Members use the bike for regular commuting perhaps to work or school and ride duration is comparetively shorter than casual users.
- Casual users ride for longer duration of time and the rides are lower on Monday through Thursday and rises on Friday reaches peak at Saturday then slight drop at Sunday.This suggests that they use the bike for sightseeing etc.
   
### Top recommendations

- 




