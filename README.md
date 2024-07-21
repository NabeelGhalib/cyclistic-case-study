# Cyclistic - A bike share analysis
# Name - Nabeel Ghalib


### Project Overview 
---

This Data Analysis project aims to provide insights into usage of the cyclistic program by the members and casual users. By Analyzing various aspects of the cycle trips data we can identify trends, make Data-driven recommendation to improve the company. 


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

### Data Analysis Phases
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

- SQL - PostgreSQL - Data Cleaning and Data Analysis.
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
uploaded all the other datasets by the above method

### Processing data (Data Cleaning)
---

- Check for null values
- Check for the formats
- Check for Duplicates
- Check for White spaces
- Check for incomplete and wrong data(spelling mistakes,irrelevant data etc.)

### Cleaning

1. I can find out the time it takes for every ride to complete by subtracting started_at from ended_at

```sql

select *, (ended_at - started_at) as ride_length
from cycle_trip_04

```
By doing this I found that there are some negative values in ride_length in all the datasets,
i.e. ended_at time was less than started_at ex- started_at - '2020-04-02 18:30:00' , ended_at - '2020-04-02 18:25:00' this is not possible so I gathered that the values were entered in wrong columns.

2. So I swapped the values of ended_at and started_at where the difference was in negative.

```sql

update cycle_trip_04
set started_at = ended_at,
	ended_at = started_at	
where (ended_at-started_at) < '00:00:00'

```
and did that for all the other datasets.

3. By looking at the datasets I know that all 6 datasets that I am going to work with have matching column names and those columns have same data types, so I combine them into one using union all

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

4. Then I checked for null values, incomplete data and spelling mistakes in all the columns.


```sql

select * from cycletripsall
where started_at is null

select * from cycletripsall
where ended_at is null

select * from cycletripsall
where ride_id is null

```
and so on.


5. Finding out the number of characters present for every ride_id


```sql

select length(ride_id) from cycletripsall

```

There are 16 characters.

6. By knowing this we can check if there are ids which does not have same no. of characters.


```sql

select ride_id from cycletripsall
where length(ride_id) != '16'

```

There were none.


7. Making ride_length and Day of the week columns


```sql

select *, (ended_at-started_at) as ride_length, to_char(started_at, 'DY') as started_at_day
from cycletripsall

```

8. Now saving the table.


```sql

create view cycletrips as
select *, (ended_at-started_at) as ride_length, to_char(started_at, 'DY') as day_started_at
from cycletripsall

```

Now the data is ready for analysis.


### Analyze
---

1. Finding  users total rides, average ride_length, minimum ride_length and maximum ride_length .
  

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


3. Finding the total number of casual users and members who ride for the following time intervals:

- Between 1 min to 10 mins
- Between 10.01 mins to 20 mins
- Between 20.01 mins to 30 mins
- Between 30.01 mins to 40 mins
- Between 40.01 mins to 50 mins
- Between 50.01 mins to 1 hour
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

![Screenshot (111)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/f06daf70-a976-47b7-b9e2-ec89d6de2f64)
![Screenshot (112)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/3d460e74-fd2e-4bf4-8ca5-626564f06ff9)
![Screenshot (113)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/b74a744c-03de-4e7d-964b-127b73edc89f)
![Screenshot (114)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/a351005f-cd4d-48da-a0e9-c71e9356136d)
![Screenshot (115)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/61481c5d-98aa-4cf8-9826-afab1f1e7a70)
![Screenshot (116)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/c08a6e2f-8ff1-4c72-bc66-8f273b0750a5)
![Screenshot (117)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/f21e4421-f6ed-473f-bdd9-28432c814eb6)
![Screenshot (118)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/5b192519-c562-46e5-b859-57e233de0a3b)
![Screenshot (122)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/515ccf3f-8687-4769-bac7-edbb98274629)
![Screenshot (119)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/54ffafc1-b092-4483-a4e6-fd7d1853fa43)
![Screenshot (120)](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/a72cd44e-a4e7-4077-ac4e-c03ee5f4b4f6)




By Analyzing the data we found out that the most users ride between these durations:
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

The most number of trips were from the following days Saturday, Sunday and Friday.


5. Now finding out the top 10 routes used by the users.

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
---

#### Now visualizing the Analyzed Data.

1.Number of rides for every day of the week for casual and member users

  
![DOWALL](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/a498d7ec-1489-4837-a000-eeca9550b1b9)



2.Number of rides for every ride duration
  

![RLCMCDashboard 1](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/d136838f-fd31-499c-82ab-8a723b1bb23d)



3.Number of rides for each month

![TRIPSMONTH](https://github.com/NabeelGhalib/cyclistic-case-study/assets/158058093/0737ae94-e198-4957-a0a3-dec87b6b9d61)



## Summary of Analysis 

- Casual users ride for 49 minutes on average.
- Members ride for 17 minutes on average.

- For Members, the most rides are in between these ride lengths 1 min - 10 mins, 10 mins - 20 mins, 20 mins - 30 mins.
- For Casual users, the most rides are in between these ride lengths 1 min - 10 mins, 10 mins - 20 mins, 20 mins - 30 mins and 1 hour - 2 hours.
  
- The most number of rides were on Saturday for both the users.
- The trips are low on Monday to Thursday, increases at Friday reaches peak on Saturday with slight drop off on Sunday.

- Members use the bikes more consistently throughout the week.
- Casual users use the bike more on Saturday, Sunday and Friday.
- Casual users ride atleast 2 times longer on any day of the week than members.

- Most of the top 10 routes of the Casual and Member users are not same.

- The most number of rides for both the users occur in the months of June, August and September where August has most number of rides.



# Act
---

How do members and casual riders use Cyclistic bikes differently?

- Members use the bike for regular commuting perhaps to work or school and ride duration is comparetively shorter than casual users.
- Casual users ride for longer duration of time and the rides are lower on Monday through Thursday and rises on Friday reaches peak at Saturday then slight drops at Sunday.This suggests that they use the bike for leisure like sightseeing etc.

   
### Top recommendations

- **Weekend only**  offer - Provide an offer at a lower price than the annual membership price where they can use the cycle for these 3 days i.e Friday, Saturday and Sunday which will attract the casual users to convert to annual member.
- **Ride duration** offer - an offer for casual users who ride for these durations 1) Between 1 min - 30 mins 2) Between 1 hour - 2 hours.
- **Top Routes** offer - an offer based on the top 10 routes used by casual users.

  






