# Cyclistic-Case-Study

### Project Overview 
This Data Analysis projects aims to provide insights into usage of the cyclistic program by the members and casual users. By Analyzing various aspects of the cycle trips data we can identify trends, make Data-driven recommendation to improve the company. 


### Scenario:
In the given scenario, I am a data analyst working in the marketing analyst team at Cyclistic,a bike share company in Chicago.
The director of marketing believes the company’s future success depends on maximizing the number of annual memberships.
Therefore, my team wants to understand how casual riders and annual members use Cyclistic bikes differently.

### Key Stakholders:

#### Primary Stakeholders:
Lily Moreno - Director of Marketing,
Cyclistic Executive Team
#### Secondary Stakeholders:
Marketing Analytics Team 

### Six Phases of Data Analysis:
#### 1.ASK
#### 2.PREPARE
#### 3.PROCESS
#### 4.ANALYZE
#### 5.VISUALIZE
#### 6.SHARE

### ASK (Business Task):
1. How do members and casual riders use Cyclistic bikes differently?
2. Why would casual riders buy Cyclistic annual memberships?
3. How can Cyclistic use digital media to influence casual riders to become members?

### Preparing Data:

### Data Sources Used:
The datasets used in this analysis have been made available by Motivate Internation Inc. 

This is public data that you can use to explore how different customer types are using Cyclistic bikes. 

Note: Cyclistic is a fictional company, hence the datasets contain the name of a different company. For the purpose of this analysis, the given datasets can be used to answer the business task 

### Tools Used:
- SQL - Postgresql - Data Cleaning and Data Analysis.
- Tableau - Data Visualization.


### Creating tables and importing the datasets: 

/* I am using 6 months of data (from 2020-04 to 2020-09) */

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

### Processing data (Data Cleaning):
- Check for null values
- Check for the formats
- Check for Duplicates
- Check for White spaces
- Check for incomplete and wrong data(spelling mistakes,irrelevant data etc.)

By Checking the datasets I know that all the 6 datasets that I am going to work with have same column names and those columns have same data types, so I combine them into one using union all

```sql

create view cycletripsall as
 
(select * from cycletrip04
union all
select * from cycletrip05
union all
select * from cycletrip06
union all
select * from cycletrip07
union all
select * from cycletrip08
union all
select * from cycletrip09)

```
Then I checked for null values,incomplete data and spelling mistakes in all the columns.

```sql

select * from cycletripsall
where started_at is null

```
and so on.

- I can find out the time it takes for every ride to complete by subtracting started_at from ended_at

```sql

select *, (ended_at - started_at) as ride_length
from cycletripsall

```
- By doing this I found at there are some negative values in ride_length , i.e. ended_at time was less than started_at ex- started_at - '2020-04-02 18:30:00' , ended_at - '2020-04-02 18:25:00' this is not possible so I assumed that the values were entered in wrong columns so I swapped the values of ended_at and started_at where the difference was in negative by doing this.

```sql

update cycletripsall
set started_at = ended_at,
	ended_at = started_at	
where (ended_at-started_at) < '00:00:00'

```
I need to know the days so i could know total trips for every day of the week.

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





