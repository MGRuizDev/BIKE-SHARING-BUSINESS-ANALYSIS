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

###Analysing the data.

<br>
<br>

#### Calculations, Metrics, and observations:

The max ride_length 
The mode of day_of_week
The average of ride_length (for members and casual riders)Try rows = member_casual; Values = Average of ride_length. 
The average ride_length for users by day_of_week. Try columns = day_of_week; Rows = member_casual; Values = Average of ride_length.
The number of rides for users by day_of_week by adding Count of trip_id to Values. 
Which bike is preferred by casual and annual members?
Which bike is used for long trips or short trips, by casual and annual members?
Which station is the busiest?
What is the average time of trip duration?
Number of trips by day, month, and year?

Exploring different seasons to make some initial observations.





### Sharing (Vizualization)

<br>
<br>
<br>
<br>

### Acting

<br>
<br>

    - What aspects of the bikes are more in demand and why?
    - Consider price comparison against competitors (being these not only bike shares but also other types of transportation: car, lift, and bus)?
    - What customers think about the bikes usability, and the app functionality. What is the feedback?
    - What is representing the largest source of profit in the business and could it be feasible to adjust?

