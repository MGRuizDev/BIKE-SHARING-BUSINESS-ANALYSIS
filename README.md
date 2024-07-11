# ELECTRIC-BIKE-SHARING-BUSINESS-ANALYSIS

<br>
<br>

### Establishing the Problem
<br>
The goal is to convert casual riders into annual members. 
There are three important aspects that can lead us to this goal: 

    * Transform annual membership into a more attractive option.
    * Firmly remark the need for casual riders to acquire an annual membership.
    * Transparency and honesty to strengthen the reputation of the business and 
      improve relations with customers.

In order to achieve these goals, we need to look into different things on the business process, 
mainly by answering the next questions we can figure out which are the main constraints that could stop 
customers committing to an annual membership.

    - What are the behaviors of the casual and annual riders and how do they contrast?
    - What aspects of the bikes are more in demand and why?
    - Consider price comparison against competitors (being these not only bike shares 
    	but also other types of transportation: car, lift, and bus)?
    - What customers think about the bikes usability, and the app functionality. What is the feedback?
    - What is representing the largest source of profit in the business and could it be feasible to adjust?

The answers for these questions will depend on data availability that might not be present at this time. 
Also, the same analysis might open new questions.

    - Which bike is preferred by casual and annual members?
    - Which bike is used for long trips or short trips, by casual and annual members?
    - Which station is the busiest?
    - What is the average time of trip duration?
    - Number of trips by day, month, and year?


<br>
<br>
<br>
<br>

### Preparation of the data.

<br>

The data comes from a reputable source. It has been made available by Motivate International Inc. 
It can be access here: <br>
https://divvy-tripdata.s3.amazonaws.com/index.html

Data was obtained in CSV format. There are 11 files: 
9 corresponding to the last 9 months of business performance and 1 of bike stations.
The data will be loaded and organized in Postgresql Database.


A description of all data sources used 

	* Information:  Trip Data
	* Type: 9 files (.csv) corresponding to the last 9 months of service.
	* Method: Manually by email from Operations manager.
	* Condition: Dirty. Duplicates, Empty values, Wrong data types, Syntax Errors.
	
	* Information: Stations Data.
	* Type: 1 file (.csv) with bike stations data from the 2014 year.
	* Method: Manually by email from Operations manager.
	* Condition: Dirty. Duplicates, Empty values, Wrong data types, Syntax Errors.

Notes:<br>
Before the data loading to database is important to remove duplicates from ride_id

<br>
<br>
<br>
<br>

### Processing the data to make it useful.

<br>

Due to the large amount of data being at least 4 million rows, 
It is not appropriate to use a spreadsheet software, 
therefore SQL is going to be used to prepare (clean and transform) the data.

I have created copies of the files, leaving the originals stored in a safe place in case their use is needed.

The process of cleaning the data are to ensure data has accuracy, 
completeness, consistency, relevance, and uniformity.

<br>

#### Data Cleaning.
<br>
Sources of errors: Did you use the right tools and functions to find the source of the errors in your dataset?
Null data: Null data was found in different columns, but specially in the primary key, which entry has to be removed.
Misspelled words: Using Group BY and observing, there were not misspells. Not Found.
Mistyped numbers: Not Found
Extra spaces and characters: Not Found
Duplicates: Using DISTINCT in SQL, there were not duplicates in primary keys or columns were are not needed. Not Found.
Mismatched data types: Not Found
Messy (inconsistent) strings: all of the strings are consistent and meaningful.
Messy (inconsistent) date formats: Dates were format correctly.
Misleading variable labels (columns): Columns named correctly.
Truncated data: Not Found
Business Logic: Data has logic.

<br>

#### Data Transformation.

Two new columns had to be created:

		* Trip_duration
		* day_of_week

In order to do that the first plan was to update the table with these two new columns,
but the query will take a long time to run. 
The second option, that was used, was to create a new table with the information of the first one,
adding these two new columns with calculations from the first table.
First table was m=named ride_trip. New table is made ride_travel.(Postgresql Query)


CREATE TABLE ride_travel AS <br>
	  SELECT ride_id,  <br>
        	rideable_type,  <br>
        	started_at,  <br>
        	ended_at,  <br>
        	start_station_name,  <br>
        	start_station_id,  <br>
        	end_station_name,  <br>
        	end_station_id,  <br>
        	start_lat,  <br>
        	start_lng,  <br>
        	end_lat, <br>
        	end_lng, <br>
        	member_casual, <br>
        	(cast (ended_at::timestamp as time) - cast (started_at::timestamp as time)) AS trip_duration, <br>
        	DATE_PART('dow', started_at) AS day_of_week <br>
FROM ride_trip;

<br>

#### Dataset design.

It consist of only one table with 15 columns and 4200000 Millions rows.


<br>
<br>
<br>
<br>

### Analysing the data.

<br>

#### Calculations, Metrics, and observations:
<br>
##### Using Postgresql.
<br>
<br>
The max ride_length. <br>
<br>
SELECT ride_id, rideable_type, started_at, member_casual, trip_duration, day_of_week FROM ride_travel <br>
WHERE trip_duration = (SELECT MAX(trip_duration) FROM ride_travel);<br>
<br>
A Sunday on April 2024, a casual user ride a electric bike for 22:47:19, being this the longest trip. <br>
<br>
<br>
The mode of day_of_week. <br>
<br>
SELECT MODE() WITHIN GROUP (ORDER BY day_of_week) AS "Most popular day" FROM ride_travel;<br>
<br>
Saturday is the most popular day to ride the bikes.<br>
<br>
<br>
The number of rides and average of ride_length for members, casual riders, and days of the week (tweak the query to analyse the next observations<br>
<br>
SELECT member_casual AS membership, day_of_week, COUNT(ride_id) AS "Number of rides", AVG(trip_duration) AS "Average trip duration" 
FROM ride_travel <br>
GROUP BY member_casual, day_of_week, rideable_type; <br>
<br>
Yearly members double the number of rides of the casual riders. 
The average trip duration of Yearly members is about 30% less than casual riders.
On weekends casual riders activity increases, and yearly riders activity reduces, despite this, yearly riders demand is more than casual riders.
<br>
You can include "readable_type" to this query and observe that casual riders prefere electric bikes and yearly members prefere classic bikes. Also, classic bikes are mostly used for long trips.
<br>
<br>
Which station is the busiest? <br>
<br>
SELECT start_station_name, COUNT(ride_id) FROM ride_travel <br>
GROUP BY start_station_name <br>
ORDER BY COUNT(ride_id) DESC; <br>
<br>
The bike stations around the River North area are the busiest.<br>
<br>
<br>
Number of trips by day, month, and year? <br>
<br>
SELECT DATE_PART('year', started_at) AS "Yearly Period", DATE_PART('month', started_at) AS "Monthly Period", COUNT(ride_id) AS "Number of rides"<br>
FROM ride_travel <br>
GROUP BY "Yearly Period", "Monthly Period"; <br>
<br>
By September the number of rides start to going down until February were the number of rides slowly start to going back up.


<br>
<br>
<br>
<br>


### Sharing (Vizualization)
<br>




<br>
<br>
<br>
<br>
[View Dashboard] (https://public.tableau.com/views/BIKESHARINGBUSINESSANALYSIS/BikeRidesBehaviorAndUsageHabits?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)
### Acting

<br>

    - What aspects of the bikes are more in demand and why?
    - Consider price comparison against competitors (being these not only bike shares but also other types of transportation: car, lift, and bus)?
    - What customers think about the bikes usability, and the app functionality. What is the feedback?
    - What is representing the largest source of profit in the business and could it be feasible to adjust?

<br>
<br>
<br>
<br>
