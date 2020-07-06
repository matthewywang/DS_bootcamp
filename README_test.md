# Metis Project 1: WomenTechWomenYes (WTWY) MTA Analysis

#### WomenTechWomenYes is interested in deploying their street teams at various NYC subway stations to collect email addresses, which they will use to send out information and tickets to their start-of-summer fundraising gala. 
#### We want to help WTWY optimize their email-collection strategy by providing them with recommendations for locations and times their street teams should operate within.
#### In our analysis, we will use MTA turnstile data, US Census data, and NYC Subway Station geographical data. MTA turnstile data will help determine high daily foot traffic stations, US Census data will provide median household income for all census tracts in NYC, and the NYC Subway Station geographical data will allow us to identify map each station to a census tract ID (and then to median household income).. 

## Identifying Key Factors

#### Weight below 3 factors using relative importance of each factor
#### Location 
Identify top stations with the most foot traffic. 
Stations with heaviest foot traffic would give WTWY street teams access to most opportunities to interact with potential attendees.
#### Demographics
Identify top stations in neighborhoods with specific income thresholds. 
The purpose of the Gala is to fundraise so it would make sense to hone in on stations located in neighborhoods with high income households.
#### Time
Optimizing date and time that the street team can focus on.
To maximize the street team’s efficiency, we wanted to track down the specific day of the week and time of the day for the team to focus their efforts.

## Packages Used

Pandas - Python package for data manipulation and analysis.

Numpy - fundamental package for scientific computing Python.

Matplotlib - plotting and visualization package for Python and numpy.

Seaborn - Python visualization package based on matplotlib.

Pickle - Python module serializing and deserializing Python objects for faster object loading

Datetime  - module allows for date and time manipulation

Requests - Python HTTP library.  Provides a simple API to abstract the complexities of making HTTP requests.

URLLIb - Package that contains several modules for working with URLs (e.g. opening and reading URLs, parsing URLs, etc.).

Fuzzywuzzy - Python library that uses Levenshtein Distance for approximate string matching.

## Datafiles Used

MTA Turnstile Data - 

http://web.mta.info/developers/turnstile.html 

All entries and exits data from MTA turnstiles from April 1st 2019 to June 30th 2019

US Census Tract Data (specifically 2017 data) - 

https://www.kaggle.com/muonneutrino/us-census-demographic-data/data

Data shows household income by US Census Tract

NYC Subway Station Geo Data (Latitude, Longitude) - 

http://web.mta.info/developers/data/nyct/subway/Stations.csv

Data gives information on Latitude and Longitude of each subway station

## EDA

### MTA data
The MTA data was the primary data source that we had to clean. Our first step was to determine the range of weeks / months to analyze. We determined that beginning of April thru end of June would be the optimal time frame to look at as the gala is for the start of summer- late-June/early-July. After reading the various weeks into a single dataframe, we cleaned the column names for whitespace and removed all duplicate entries by only keeping unique “C/A", "UNIT", "SCP", "STATION", "DATE_TIME" combinations. We removed all non-consecutive days and separated stations (in the “STATION” column) that seemed to be accumulating data from multiple physical, geographical locations by making an assumption that if a control area did not have any common train lines, it represented a separate location. We named these separated stations by their unique train lines.

In order to get our daily traffic value, we first sorted the data by “C/A", "UNIT", "SCP", "STATION", "DATE_TIME" in ascending order.  Then we shifted our data down a row as we needed to track the previous day’s entries and exits. We removed data that seemed erroneous (e.g. turnstile ticker resetting), greater than 1 person coming in per second for the entire day (as an initial assumption).  We also adjusted the data for a turnstile ticker decrementing instead of incrementing. In order to further remove outliers in our daily traffic, we compared the max value to the median value (defined as “peak_factor”) and removed rows where the value appeared to be an outlier based on its size relative to the max and median values. In case any outlier values had still slipped through, we further removed rows where any values (daily entries, daily exits) that were greater than 5 times the median (daily entries, daily exits for that “C/A”, “UNIT”, “SCP”, “STATION” combination). We then reset the shift we had made earlier in order to properly track the date on which the traffic took place (since we had sorted ascending order, and most date timestamps started at or close to midnight 00:00:00).  Following this, we added up daily entries and daily exits to arrive at a “total traffic” number by turnstile, station, date.

### US Census Tract Data
First we filtered the data to only the five boroughs of New York City relevant for this project.  Then we calculated the household income percentile for each census tract ID, and multiplied those by 10 to create a “score.”

### Merging Dataframes
We used the function df.merge() to merge our turnstile, census tract, and subway geo data.

For turnstile and census tract dataframes, we merged on ‘Tract ID’.

For subway geo data and turnstile data, we wanted to merge on “Station Name” but found that many of the station names between these two datasets were mismatched in both spelling and naming structure. As such, we had to incorporate fuzzy matching and then merge the data frames on the fuzzy-matched station names.

## Creating Visualizations

#### Top 5 stations with the highest daily foot traffic
After cleaning the MTA turnstile data, we aggregated total traffic (cumulative sum) by station name and sorted in descending order to find the top 5 stations.

#### Median HH income of the top 5 stations


#### Income percentile for the top 5 stations


#### Total traffic by day of week for the top 5 stations


#### Traffic heat map for the busiest hour of the day


### Analysis and Recommendation
We used our 3 factors framework in order to conduct our analysis. As stated above, our 3 factors were time, location, and income demographics.

For location, we found the top 5 stations in terms of total foot traffic (entries + exits), and discovered that the total traffic for each of these stations exceeded 15 million - 10x more than the median traffic per station.

For income, we found that the tracts surrounding the top 5 trafficked stations also closely reflected the top 5 stations with highest income. 14th Street Penn Station, 24th Street Herald Square, and 14th Street Union Station in particular were all in tracts with household incomes exceeding $140,000.

For time, we filtered our data according to the top 5 stations above and found that of those, Tuesday, Wednesday, and Thursday during the 4-8pm block constituted the highest-trafficked times.

## Authors

Matt Wang, Louis Sagan, Preston Lam, Sophia Li

#### Thanks for reading!

<img src="https://4.bp.blogspot.com/-6qFetQ3Mti4/VcosbirJeaI/AAAAAAAAdXs/Rg9Rl9ANbro/s1600/IMG_2892%2B%25281%2529.jpg" width="550">

