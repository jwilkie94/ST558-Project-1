ST558 Project 1
================
Jenna Wilkie
10/2/2021

## Requirements

I used the following packages to interact with the NASA API.

*tidyverse*: A package useful for data manipulation. Installation of the
*tidyverse* package also includes the *readr*, *tidyr*, *dplyr*, and
*tibble* pakages.  
*jsonlite*: A library containing the function used to connect to the API
through URL.  
*magick*: A library used to save image from API URL.  
*chron*: A library that allows for date conversions included hours and
minutes. *RCurl*: A library that contains the getURL function that will
be used to obtain the API URL.

# Functions

## *DateConv* Helper Function

This function was created to convert multiple date formats into the
standard format used for the API endpoint URL’s. This will allow the
user to enter the date in a variety of formats.

``` r
DateConv<-function(date){
 date<-as.Date(date, tryFormats=c("%m-%d-%y","%m-%d-%Y","%m/%d/%y", "%m/%d/%Y",   "%B %d %Y", "%Y-%m-%d", "%Y/%m/%d", "%B %d, %Y","%b %d, %Y", "%b %d %Y", "%B %d %y", "%B %d, %y", "%b %d %y", "%b %d, %y" ),            optional=TRUE)
 return(date)
}
```

## *Imagery* Function

This function is used to provide the URL to access the Landsat 8 image
for a specific location and date.

``` r
Imagery<-function(latitude, longitude, date, api_key){
 #convert date to account for multiple formats
 date<-DateConv(date)
 
 #get the data from the API
 outputAPI<-paste0("https://api.nasa.gov/planetary/earth/imagery?lon=",longitude, "&lat=", latitude, "&date=",  date,"&dim=0.15","&api_key=",api_key)
 
 #Return webpage with image
 print(image_read(outputAPI))
}
```

## *Assets* Function

This function is used to access the date times and asset names for the
closest available imagery.

``` r
Assets<-function(latitude, longitude, date, api_key){
 #convert date to account for multiple formats
 date<-DateConv(date)
 
 #get the data from the API
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/planetary/earth/assets?lon=",longitude, "&lat=",latitude,"&date=",date,"&dim=0.15","&api_key=",api_key)))
 
outputAPI
}
```

## *Neo-Feed* Function

This function was created to gather a list of asteroids between two
dates up to seven days apart and lists them based on their closest
approach to earth. Each Date will be returned as a data frame of
asteroids.

``` r
NeoFeed<-function(start_date, end_date, api_key){
 #convert the start and end dates to the appropriate format
 start_date<-DateConv(start_date)
 end_date<-DateConv(end_date)
 
 #get the data from the API
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/neo/rest/v1/feed?start_date=",start_date,"&end_date=",end_date,"&api_key=",api_key)))
 
 #return data frames
 outputAPI
}
```

## *Neo-Lookup* Function

This function was created to allow the user to look up an asteroid based
on its NASA ID.

``` r
NeoLookup<-function(asteroid_id, api_key){
 #get the API
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/neo/rest/v1/neo/",asteroid_id,"?api_key=",api_key)))
 
outputAPI
}
```

## *Weather* Function

This function allows the user to look up certain space weather events
based on date and type of event.

``` r
Weather<-function(start_date,end_date,event,api_key){
#convert the start and end dates to the appropriate format 
 start_date<-DateConv(start_date)
 end_date<-DateConv(end_date)

 #allow for multiple event entries to make the function more user friendly.  Must convert event entry to all lower case letters first to account for differences in capitalization.  
event<-tolower(event)

if (event=="coronal mass ejection"||event=="cme"||event=="coronal mass ejection (cme)"){
event<-"CME"
}
else if (event=="geomagnetic storm"||event=="gst"||event=="geo storm"||event=="geomagnetic storm (gst)"){
event<-"GST"
}
else if (event=="solar flare"||event=="flr"||event=="flare"||event=="solar flare (flr)"){
event<-"FLR"
}
else if (event=="solar energetic particle"||event=="sep"||event=="solar energetic particle (sep)"){
event<-"SEP"
}
else if (event=="magnetopause crossing"||event=="mpc"||event=="magnetopause crossing (mpc)"){
  event<-"MPC"
}
else if (event=="radiation belt enhancement"||event=="rbe"||"radiation belt enhancement (rbe)"){
  event<-"RBE"
}
else if (event=="hight speed stream"||event=="hss"||event=="hight speed stream (hss)"){
  event<-"HSS"
}
else {stop("Error: Invalid weather event entry!")}

#get API data
outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/DONKI/",event,"?startDate=",start_date,"&endDate=",end_date,"&api_key=",api_key)))


outputAPI
}
```

## Techport Function

This function allows the user to make queries about NASA’s technology
development programs.

``` r
Techport<-function(parameter_id, api_key){
 #get API data
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/techport/api/projects/",id_parameter,"?api_key=",api_key)))

outputAPI
}
```

## NASAAPI Wrapper Function

This function combines the API searching functions above into one, to
allow the user to input the function and arguments at once.

``` r
NASAAPI<-function(func, ...){
if (func == "Imagery"){
    output <- Imagery(...)
}
else if (func == "Assets"){
    output <- Assets(...)
}
else if (func == "NeoFeed"){
    output <- NeoFeed(...)
}
else if (func == "Neo-Lookup"){
    output <- NeoLookup(...)
}
else if (func == "Weather"){
    output <- Weather(...)
}
else if (func == "Techport"){
    output <- Techport(...)
}
else {stop("Error: Invalid function entry!")}
}
```

# Exploratory Data Analysis

# EDA: Weather Function

Now that the functions are ready, I can do some data exploration. I am
going to start with the Weather function, and pull data from solar
flares that occured in
2019.

``` r
#use the NASAAPI function to pull solar flare events in 2019 and save the output as an object.
SolarFlare<-NASAAPI("Weather", "Jan 1 2019", "Dec 31 2019", "solar flare","zMTdCaPaYeIjYgd9N91EsaFUvxYsCMR1o32ih13X")
SolarFlare
```

    ##                         flrID             instruments         beginTime          peakTime endTime classType
    ## 1 2019-01-26T13:13:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-01-26T13:13Z 2019-01-26T13:22Z      NA      C5.0
    ## 2 2019-01-30T05:57:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-01-30T05:57Z 2019-01-30T06:11Z      NA      C5.2
    ## 3 2019-03-08T03:07:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-03-08T03:07Z 2019-03-08T03:18Z      NA      C1.3
    ## 4 2019-03-20T07:05:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-03-20T07:05Z 2019-03-20T07:14Z      NA      B6.1
    ## 5 2019-03-20T10:35:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-03-20T10:35Z 2019-03-20T11:18Z      NA      C4.8
    ## 6 2019-03-21T03:08:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-03-21T03:08Z 2019-03-21T03:12Z      NA      C5.6
    ## 7 2019-05-06T05:04:00-FLR-001 GOES15: SEM/XRS 1.0-8.0 2019-05-06T05:04Z 2019-05-06T05:10Z      NA      C9.9
    ##   sourceLocation activeRegionNum                linkedEvents
    ## 1         N05W26           12733                        NULL
    ## 2         N05W79           12733                        NULL
    ## 3         N09W04           12734 2019-03-08T04:17:00-CME-001
    ## 4         N09W23           12736 2019-03-20T08:24:00-CME-001
    ## 5         N08W26           12736 2019-03-20T08:24:00-CME-001
    ## 6         N08W34           12736                        NULL
    ## 7         N08E50           12740                        NULL
    ##                                                       link
    ## 1 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14440/-1
    ## 2 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14462/-1
    ## 3 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14548/-1
    ## 4 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14570/-1
    ## 5 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14571/-1
    ## 6 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14577/-1
    ## 7 https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/FLR/14704/-1

For the purposes of this analysis, I want to modify the character string
containing the start times and peak times. First I will remove the “T”
and “Z” so only the date and time are returned, then I will convert
those into chron values that can be used in
calculations.

``` r
#remove the additional information from the start times for CME's and Solar Flares and convert the string into date format including hours and minutes
 a<-substr(SolarFlare$peakTime, 1, 10) #substring containing the date only of peak
 b<-substr(SolarFlare$peakTime, 12, 16) #substring containing the hours and minutes of peak
 c<-paste(a,b) #combine substrings into one character string 
 d<-substr(SolarFlare$beginTime, 1, 10) #substring containing the date only of begin
 e<-substr(SolarFlare$beginTime, 12, 16) #substring containing the hours and minutes of begin
 f<-paste(d,e) #combine substrings into one character string

 #convert both strings to chron format and replace existing variables
SolarFlare$beginTime<-as.chron(c, format="%Y-%m-%d  %H:%M") 
SolarFlare$peakTime<-as.chron(f, format="%Y-%m-%d  %H:%M")
```

Now that the data has been converted into a chron format, I can do
calculations with the dates. I want to start by creating a variable in
the solar flare data set that measures the time difference between the
beginning time and the peak of each solar flare event (in
seconds).

``` r
SolarFlare<-SolarFlare %>% mutate(begin_peak=difftime(beginTime,peakTime,units="secs"))
```

Now that I have the difference time, I want to create a categorical
variable that separates the begin\_peak variable into groups by length
of time difference: short (\<1000 seconds), medium (1000 to 2000
seconds), and long(\>2000
seconds).

``` r
SolarFlare<-SolarFlare %>% mutate(length=if_else(begin_peak>2000, "Long", if_else(begin_peak >=1000, "Medium","Short")))
```

With this final data frame, I can create a contingency table of the
length and Active Region Number of the flare. The table shows us that
there are no medium length flares in the data set, and only 1 long
flare.

``` r
table(SolarFlare$activeRegionNum,SolarFlare$length)
```

    ##        
    ##         Long Short
    ##   12733    0     2
    ##   12734    0     1
    ##   12736    1     2
    ##   12740    0     1

I can also create a barplot of these
results.

``` r
#convert the active region number to a character variable in the ggplot statement.  
bp<-ggplot(SolarFlare, aes(x=as.character(activeRegionNum)))
bp+geom_bar(aes(fill=as.factor(length)), position="dodge")+labs(x="Active Region Number", y="Count", title="Bar Plot of Solar Flare Events per Region by Length")+scale_fill_discrete(name="Length")
```

![](README_files/figure-gfm/barplot1-1.png)<!-- -->

## EDA: NeoFeed Function

Next I will look at data from the NeoFeed function. I want to return
data frames containing asteroid information for 3/27/2020 and 3/28/2020.
The function returns a list of data frames; one for each date in the
search
range.

``` r
NeoF<-NASAAPI("NeoFeed", "03/27/2020", "03/28/2020", "zMTdCaPaYeIjYgd9N91EsaFUvxYsCMR1o32ih13X")
```

I am going to combine some columns into a new data frame. I am
interested in the absolute magnitude’s relation to estimated diameter
(in meters) and whether the asteroid is potentially hazardous or not. To
explore this further, I will create a new data frame with those columns
and aolumn for the date.

``` r
#create a new column "date" in each data frame 
NeoF$near_earth_objects$`2020-03-27`$date<-"2020-03-27"
NeoF$near_earth_objects$`2020-03-28`$date<-"2020-03-28"
```

``` r
#combine the desired columns from each data frame
NeoNew1<-cbind(NeoF$near_earth_objects$`2020-03-27`$absolute_magnitude_h, NeoF$near_earth_objects$`2020-03-27`$estimated_diameter$meters$estimated_diameter_min, NeoF$near_earth_objects$`2020-03-27`$estimated_diameter$meters$estimated_diameter_max, NeoF$near_earth_objects$`2020-03-27`$is_potentially_hazardous_asteroid, NeoF$near_earth_objects$`2020-03-27`$date)

NeoNew2<-cbind(NeoF$near_earth_objects$`2020-03-28`$absolute_magnitude_h, NeoF$near_earth_objects$`2020-03-28`$estimated_diameter$meters$estimated_diameter_min, NeoF$near_earth_objects$`2020-03-28`$estimated_diameter$meters$estimated_diameter_max, NeoF$near_earth_objects$`2020-03-28`$is_potentially_hazardous_asteroid,NeoF$near_earth_objects$`2020-03-28`$date)

#combine data from both dates into one final data frame and add column names 
NeoNew<-rbind(NeoNew1, NeoNew2)
colnames(NeoNew)<-c('Absmag', 'MinDia', 'MaxDia', 'potential_hazard', 'date')

#convert to tibble
NeoNew<-as.tibble(NeoNew)

#convert all columns containing numbers to integer format
NeoNew$Absmag<-as.integer(NeoNew$Absmag)
NeoNew$MinDia<-as.integer(NeoNew$MinDia)
NeoNew$MaxDia<-as.integer(NeoNew$MaxDia)
```

Now that the necessary data is combined, I can produce a contingency
table for date and the potential hazard of the asteroid.

``` r
Con2<-table(NeoNew$potential_hazard, NeoNew$date)
Con2
```

    ##        
    ##         2020-03-27 2020-03-28
    ##   FALSE         16         13
    ##   TRUE           1          0

Only 1 asteroid was potentially hazardous on March 27, 2020 and none
were potentially hazardous on March 28, 2020 (lucky for us\!).

Next I will look at how the maximum estimated diameter compares to the
minimum. Below is a scatter plot between the minimum estimated diameter
and maximum estimated diameter of each asteroid.

``` r
sp<-ggplot(NeoNew, aes(x=MinDia, y=MaxDia))
sp+geom_point(aes(col=date))+labs(title="Min vs. Max Estimated Diameter", x='Minimum Estimated Diameter', y='Maximum Estimated Diameter' )
```

![](README_files/figure-gfm/scatter-1.png)<!-- -->

The correlation looks to be close to 1, so the graph shows a strong
linear relationship. Date does not appear to affect the correlation, but
March 27, 2020 had larger asteroids approaching.

I have two values for estimated diameter, max and min. I am going to add
a variable to the new data frame that is the approximate average
estimated diameter of each asteroid.

``` r
NeoNew$MeanDia<-(NeoNew$MaxDia+NeoNew$MinDia)/2
```

Now, I can graph the absolute magnitude against mean estimated diameter
and find the correlation.

``` r
#find the correlation of the data
correlation<-cor(NeoNew$MeanDia,NeoNew$Absmag)
sp2<-ggplot(NeoNew, aes(x=MeanDia, y=Absmag))
sp2+geom_point(aes(col=date))+labs(title="Mean Estimated Diameter vs. Absolute Magnitude", x='Mean Estimated Diameter', y='Absolute Magnitude')+geom_text(x=250, y=18,label=paste0("Correlation= ",round(correlation,2)))
```

![](README_files/figure-gfm/scatter2-1.png)<!-- -->

The correlation is fairly strong (-.82), but the graph looks like it has
a curve, suggesting a possible non linear relationship between the two
variables. The larger asteroids on 3-27 are making the curve more
pronounced.

Next I’ll look at a boxplot of the absolute magnitude data. I want to
group this by date as well.

``` r
bp<-ggplot(NeoNew,aes(x=date, y=Absmag))
bp+geom_boxplot(fill='purple')+labs(title="Boxplot of Absolute Magnitude by Date", y="Absolute Magnitude", x="Date")
```

![](README_files/figure-gfm/box-1.png)<!-- -->

March 28, 2020 has larger absolute magnitudes than March 27, 2020. The
scatter plot above shows that the opposite is true for minimum and
maximum estimated diameter. This observation agrees with the negative
correlation observed in the scatter plot above.

Last, I will look at a histogram for absolute magnitude.

``` r
hp<-ggplot(NeoNew,aes(x=Absmag))
hp+geom_histogram(color='grey',fill='turquoise', binwidth=2)+labs(title='Frequency of Absolute Magnitude Count', x='Absolute Magnitude', y='Count')
```

![](README_files/figure-gfm/histogram-1.png)<!-- -->

The counts for absolute magnitude appear to follow an approximately
normal distribution, so there were not a lot of unusually large or
unusually small magnitudes observed on these two dates.
