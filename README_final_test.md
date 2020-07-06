# Metis Project 1: WomenTechWomenYes (WTWY) MTA Analysis

#### WomenTechWomenYes is interested in deploying their street teams at various NYC subway stations to collect email addresses, which they will use to send out information and free tickets to their start-of-summer fundraising gala. 
#### We want to help WTWY optimize their email-collection strategy by providing them with recommendations for locations and times their street teams should operate within.
#### In our analysis, we collected and analyzed MTA turnstile data, US Census data, and NYC Subway Station geographical data.  
MTA turnstile data will help determine high daily foot traffic stations.
US Census data will provide median household income for all census tracts in NYC.
NYC Subway Station geographical data will allow us to map each station to a census tract ID (and then to median household income).

## 1. Identifying Key Factors

#### Weight below 3 factors using relative importance of each factor
#### Location 
Identify top stations with the most foot traffic. 
Stations with heaviest foot traffic would give WTWY street teams access to most opportunities to interact with potential attendees.
#### Demographics
Identify top stations in neighborhoods with specific income thresholds. 
The purpose of the Gala is to fundraise so it would make sense to hone in on stations located in neighborhoods with high income households.
#### Time
Optimizing date and time that the street team can focus on.
To maximize the street team’s efficiency, we wanted to track down the specific days of the week and times of the day for the team to focus their efforts.

## 2. Packages Used

- pandas: Python package for data manipulation and analysis.
- numpy: fundamental package for scientific computing in Python
- matplotlib: plotting and visualization package for Python
- seaborn: Python visualization package based on matplotlib
- pickle: Python module serializing and deserializing Python objects for faster object loading
- datetime: module allows for datetime extraction and manipulation
- requests: Python HTTP library that provides a simple API to abstract the complexities of making HTTP requests
- urllib: package that contains several modules for working with URLs (e.g. opening and reading URLs, parsing URLs, etc.)
- fuzzywuzzy: Python library that uses Levenshtein Distance algorithm for approximate string matching.

## 3. Datafiles Used

MTA Turnstile Data 
http://web.mta.info/developers/turnstile.html 
All entries and exits data from MTA turnstiles from April 1st 2019 to June 30th 2019

US Census Tract Data (specifically 2017 data)
https://www.kaggle.com/muonneutrino/us-census-demographic-data/data
Data shows household income by US Census Tract

NYC Subway Station Geo Data (Latitude, Longitude)
http://web.mta.info/developers/data/nyct/subway/Stations.csv
Data gives information on latitude and longitude of each subway station

## 4. EDA

### 4a. MTA data
The MTA data was the primary data source that we had to clean.  Our first step was to determine the range of weeks / months to analyze.  We determined that beginning of April thru end of June would be the optimal time frame to look at as the gala is for the start of summer (late June / early July).  After reading the various weeks into a single dataframe, we cleaned the column names for whitespace and removed all duplicate entries by only keeping unique “C/A", "UNIT", "SCP", "STATION", "DATE_TIME" combinations.  We removed all non-consecutive days and separated stations (in the “STATION” column) that seemed to be accumulating data from multiple physical, geographical locations by making an assumption that if a control area did not have any common train lines, it represented a separate location.  We named these separated stations by their unique train lines.

In order to get our daily traffic value, we first sorted the data by “C/A", "UNIT", "SCP", "STATION", "DATE" in ascending order.  Then we shifted our data down a row as we needed to track the previous day’s entries and exits.  We removed data that seemed erroneous (e.g. turnstile ticker resetting), i.e. greater than a single person coming in per second for the entire day (as an initial assumption).  We also adjusted the data for a turnstile ticker decrementing instead of incrementing, as well as records with date differences greater than a single day.  In order to further remove outliers in our daily traffic, we compared the max value to the median value (defined as “peak_factor”) and removed rows where the value appeared to be an outlier based on its size relative to the max and median values.  In case any outlier values had still slipped through, we further removed rows where any values (daily entries, daily exits) that were greater than 5 times the median (daily entries, daily exits for that “C/A”, “UNIT”, “SCP”, “STATION” combination). We then reset the shift we had made earlier in order to properly track the date on which the traffic took place (since we had sorted ascending order, and most date timestamps started at or close to midnight 00:00:00).  Following this, we added up daily entries and daily exits to arrive at a “total traffic” number by turnstile, station, date.

### 4b. US Census Tract Data
First we filtered the data to only the five boroughs of New York City relevant for this project.  Then we calculated the household income percentile for each census tract ID, and multiplied those by 10 to create a “score” (out of 10).

### 4c. NYC Subway Station Geo Data (Latitude, Longitude)
After importing urllib and requests, we used an API from FCC website to determine the census tract ID for each station / latitude / longitude combination (reverse geomapping). 

### 4d. Merging Dataframes
We first merged subway geo data with census tract data on “Tract ID” to get the household income scores for each subway station.  Then we merged that dataframe with the MTA turnstile data dataframe using fuzzy matching based on approximate matches for Station name, with some manual checking and a few changes.

Finally, we gave a 50 / 50 weight to the traffic score and the median household income score and gave our final recommendation of top five stations with the highest weighted scores.  We then drilled down into the MTA data for those specific stations, tracking the daily traffic by day of week.  After finding Thursday, Wednesday, and Tuesday to be the top three days with the most total volume of entries and exits, we filtered the data further to take a deeper dive into those three days.

Finally, we grouped the data by trailing four-hour periods (dropping erroneous data) and created a heatmap to discover the best four-hour windows to recommend to WTWY.  The period from 4pm-8pm clearly had higher traffic than any other window, and we found that 8am-12pm and 12pm-4pm were relatively comparable.  We made one last assumption that people are more likely to provide an email address during or after their lunch break than in the morning and therefore ranked the 12pm-4pm window above the 8am-12pm period.

## 5. Creating Visualizations

#### Top 20 stations with the highest daily foot traffic
After cleaning the MTA turnstile data, we aggregated total traffic (cumulative sum of entries and exits) by station name and sorted in descending order to find the top 20 stations

#### Median HH income of the top 20 stations
Cleaned the US census data and filtered the top 20 stations with highest foot traffic and graphed their median HH income

#### Income percentile for the top 20 stations
Used the same data as above and instead of using median HH income, we graphed top 20 stations’ income percentile (based on the census tract ID of each station)

#### Total traffic by day of week for the top 5 stations
FIltered the MTA data to include only the top 5 stations based on 50 / 50 weighted score of foot traffic and income and identified days of the week with the most traffic

#### Traffic heatmap for time of day for top 5 stations and top 3 days of week
Created a chart to show which trailing 4-hour periods hours were the busiest for our top 5 stations for top three days of the week

## 6. Analysis and Recommendation
We used our three factors framework in order to conduct our analysis.  As stated above, our three factors were time, location, and income demographics.

For location, we found the top five stations in terms of total foot traffic (entries plus exits), and discovered that the total traffic for each of these stations exceeded 15 million, which was over 10x more than the median traffic per station.

For income, we found that a wide variety of median household income for the census tracts in which the top 20 most trafficked stations are located.  In particular, although 59 St Columbus Circle and 47-50 Sts - Rockefeller Center were not in the top five stations by total traffic, their very high rank in household incomes elevated them to be included in our top five final recommendations (see below).

Based on our weighted score methodology (50% traffic, 50% income), we recommend the following top 5 stations:
   1. 34 St - Penn Station
   2. 59 St - Columbus Circle
   3. 34 St - Herald Square
   4. 14 St - Union Sq
   5. 47-50 Sts - Rockefeller Center

For time of day, we filtered our data according to the top five stations above and found that of those, Tuesday, Wednesday, and Thursday during the 4-8pm block constituted the optimal day/time windows for canvassing and collecting email addresses.

## 6. Authors

Matt Wang, Louis Sagan, Preston Lam, Sophia Li

#### Thanks for reading!

<img src="https://4.bp.blogspot.com/-6qFetQ3Mti4/VcosbirJeaI/AAAAAAAAdXs/Rg9Rl9ANbro/s1600/IMG_2892%2B%25281%2529.jpg" width="550">