# BellaBeat-casestudy-with-SQL
This is my first case study which I do it because it's part of Google Data Analytics Professional Certificate.
# introduction
 Bellabeat is a successful small company, but they have the potential to become a larger player in the
global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart
device fitness data could help unlock new growth opportunities for the company. I have been asked to focus on one of
Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The
insights you discover will then help guide marketing strategy for the company. I will present my analysis to the Bellabeat
executive team along with your high-level recommendations for Bellabeat’s marketing strategy.
# 1. ask phase
I need to find out by using the data:

 1.What are some trends in smart device usage?

2. How could these trends apply to Bellabeat customers?

3. How could these trends help influence Bellabeat marketing strategy?

# 2.Prepare phase
first I download the data set from here https://www.kaggle.com/arashnic/fitbit

This Kaggle data set contains personal fitness tracker from thirty fitbit users for 31 days.
The data is public and is open source.
### data Organisation
There are 18 datasets provided from the link mentioned above Each datasets represents a different quantitative data tracked by Fitbit device by Day, by hour or by minute.

I stored the data at dataset called "BellaBeat_c2".
### data limitation

1.data is just about 33 user only, and it's kind a small data sample.

2.it's cover a small period of time.

3.missing some demographic data like location and the age which they can improve our analysis.
# 3.Process phase
### Tools Used

I would like to use

1. "Microsoft SQL Server" to querying the data. 

2. "Tableau Public" to visualize the result.

3. "Microsoft excel" for some calculation processing.

Now I used SQL Server Import and Export Data (64-bit) to import the data

to start processing the data .

### primary key

I will check for the column name share across the table

```
SELECT
        COLUMN_NAME,
        COUNT(COLUMN_NAME) 
FROM 
        BellaBeat_c2.INFORMATION_SCHEMA.COLUMNS
GROUP BY
        COLUMN_NAME 
order by
        count(COLUMN_NAME) desc 

```

I found there are 18 "Id" column

### Data cleaning

by checking the SCHEMA of every table I found every data type of the columns is varchar(50),
so I need to convert the data type every time I want to make a calculation or use an aggregate function

![datatype](https://user-images.githubusercontent.com/41892582/180022344-dcd048e2-a4a4-450a-92e7-84ba0e22c533.jpg)

checking for missing values

```
select
       'dailyActivity_Ids',count(distinct(Id))
from
        BellaBeat_c2..dailyActivity_merged 
union

select
        'heartrate_Ids',count(distinct(Id))
from
         BellaBeat_c2..heartrate_seconds_merged 
union
select
        'sleep_Ids', count(distinct(Id))
from 
        BellaBeat_c2..sleepDay_merged 
union
select
       'weight_Ids',count(distinct(Id))
from
       BellaBeat_c2..weightLogInfo_merged 

```
I found the 33 users has a data for daily activities, 24 for the sleeps,14 for heart rate, and only 8 for the weight

```
dailyActivity_Ids	33
heartrate_Ids	        14
sleep_Ids	        24
weight_Ids	         8
```


### Understanding the Data

summarizing the data to understand it

```
select 
       count(distinct(Id)) as Ids,
       avg(cast(TotalDistance as float)) as average_of_total_distance,
       avg(cast(calories as int)) as calories

from 
       BellaBeat_c2..dailyActivity_merged

select 
       avg(cast(LightlyActiveMinutes as int)) as average_of_lightly_active_minutes,
       avg(cast(FairlyActiveMinutes as int)) as average_fairly_active_minutes,
       avg(cast(VeryActiveMinutes as int)) as average_of_very_active_minutes
from
       BellaBeat_c2..dailyIntensities_merged

select
       avg(cast(StepTotal as int)) as daily_average_steps,
       max(cast(StepTotal as int))as maximum_daily_steps,
       min(cast(StepTotal as int))as minimum_daily_steps
from
       BellaBeat_c2..dailySteps_merged

select 
       (avg(cast(totalminutesasleep as float))/60) as average_daily_sleep,
       (max(cast(totalminutesasleep as float))/60) as maximum_daily_sleep,
       (min(cast(totalminutesasleep as float))/60) as minimum_daily_sleep
from
       BellaBeat_c2..sleepDay_merged

```

###### The summary result

```
Ids	average_of_total_distance   calories
33	5.48970212191542	    2303
```
```
average_of_lightly_active_minutes   average_fairly_active_minutes   average_of_very_active_minutes
192	                            13	                            21
```
```
daily_average_steps	maximum_daily_steps	minimum_daily_steps
7637	                36019           	0
```

```
average_daily_sleep	maximum_daily_sleep	minimum_daily_sleep
6.9911218724778 	13.2666666666667	0.966666666666667
```
# 4.Analyze phase

### Data exploration and visualization

first,I need to check how often the device in using during the day.

I creating a temp table for device usage per day.

```
drop table if exists BellaBeat_c2..#device
create table #device (Id varchar(20),device_usage_by_min float)
insert into #device

select
  Id,cast(VeryActiveMinutes as float)+cast(FairlyActiveMinutes as float)+cast(LightlyActiveMinutes as float)+cast(SedentaryMinutes as float)
from
         BellaBeat_c2..dailyActivity_merged
```
find the percentage of using the device per day
```
select
        Id,((avg(device_usage_by_min))/ 1440)*100 as percentage_d
from
        #device
group by
        Id
order by
        percentage_d desc
```

visualizing this data using Tableau 

![percentage of using the device per day](https://user-images.githubusercontent.com/41892582/179403434-a523acc5-ade4-4383-beab-31898216d1ea.jpg)


calculate the total steps and average steps per day for every member.
```
select
         Id,sum (cast(TotalSteps as int)) as 'Total_steps', avg(cast(TotalSteps as int)) as avreage_per_day
from
         BellaBeat_c2..dailyActivity_merged
group by
         Id
order by
         Total_steps desc
```

plotting the average steps per every member
![average steps per day for every member](https://user-images.githubusercontent.com/41892582/179403417-8876b812-112f-49e0-b712-911f99a0e88f.jpg)

Average minutes by every type of activities
```
select
         Id,
		 avg(cast(VeryActiveMinutes as int)) as average_of_VeryActiveMinutes ,
		 avg(cast(FairlyActiveMinutes as int)) as average_of_FairlyActiveMinutes,
		 avg(cast(LightlyActiveMinutes as int)) as average_of_LightlyActiveMinutes,
		 avg(cast(SedentaryMinutes as int)) as average_of_SedentaryMinutes
		
from
         BellaBeat_c2..dailyActivity_merged
group by
         Id
```
![Average minutes per activities2](https://user-images.githubusercontent.com/41892582/179530792-f5073839-e5a9-48a8-a5e1-c354637b1a61.jpg)

Average of steps by every hour during the day
```
select
        distinct(cast(ActivityHour as time)) as The_hour,
	avg(cast(StepTotal as int)) as avg_steps_per_hour
from
         BellaBeat_c2..hourlySteps_merged
group by 
         cast(ActivityHour as time)
order by 
         cast(ActivityHour as time)

```

visualizing this data using Tableau 

![the_hour](https://user-images.githubusercontent.com/41892582/180209704-b5cb7769-b5ae-4886-8bd0-df4451c0d1a6.jpg)


I will try to find if there is a correlation between total steps during the day and the time in bed before sleeping

adding a column to the sleeping table to find the time in bed before sleep

and add the day column to join with the total steps table
```
alter table
            BellaBeat_c2..sleepDay_merged
add
            the_day varchar(30),time_in_bed_before_sleep float
update
            BellaBeat_c2..sleepDay_merged
set
            the_day=trim(SUBSTRING(SleepDay,1,9)),
            time_in_bed_before_sleep=(cast(TotalTimeInBed as float)-cast(TotalMinutesAsleep as float))
```
join two table to find the correlation coffection
```
select
       StepTotal,time_in_bed_before_sleep
from
       BellaBeat_c2..dailySteps_merged
inner join
        BellaBeat_c2..sleepDay_merged
on
       dailySteps_merged.Id=sleepDay_merged.Id and
       dailySteps_merged.ActivityDay=sleepDay_merged.the_day
```
by using "Microsoft excel" I found the correlation coefficient equal 0.027

I will try to find if there a correlation between sedentary time and total time sleep

```
select
        sedentaryminutes,totalminutesasleep
from
        BellaBeat_c2..dailyActivity_merged
inner join
        BellaBeat_c2..sleepDay_merged
on 
        dailyActivity_merged.Id=sleepDay_merged.Id
and
        dailyActivity_merged.ActivityDate=sleepDay_merged.the_day
order by
        SedentaryMinutes

```

after deleting the first 4 rows when the sedentary minutes equal 0,0,2 and 2 which make no sense.

![totalminutesasleep vs  sedentaryminutes](https://user-images.githubusercontent.com/41892582/180262375-b25ddae5-8459-4806-81bb-28d9a11cd8c5.png)

by using "Microsoft excel" I found the correlation coefficient equal -0.64

### conclusion of the analysis

1. some users leave the device far from them during the day (around half of them leave it for 4-6 hours). 
2. 75 percent of the users took the average of total steps per day less than 10000 steps works out to approximately five miles That’s a number said to help reduce certain health conditions, such as high blood pressure and heart disease according to CDC’s recommendation.
3. the most active hours during the day are between 4 and 7 pm.
4. No correlation was found between time spent in the bed before sleep and average steps during the day.
5. there is a strong inverse correlation between the night's sleep and sedentary minutes. according to the National Library of medicine "Longer time spent in sedentary posture is significantly associated with higher CHD risk and larger waist circumference"

# 6. Act phase

### recommendations and insights
1. Let the users enter some data manually like the weight and sleep time to improve the results since they didn't wear the devices all the time
2. encourage the users to take more steps during the day by making daily and weekly challenges and let them to collect badges for completing those challenges and share them with others
3. make a weekly summary and compare it with the previous week and notificate the users with it to inform them how they improve they activities
4. notificate the users with they sleep patterns, and who didn't sleep well inform them to try to decrease the time of sedentary during the day
