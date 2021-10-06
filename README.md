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
*Rcurl*: A library used to read in URL into JSON format *chron*: A
library that allows for date conversions included hours and minutes.

\#Functions

## *DateConv* Helper Function

This function was created to convert multiple date formats into the
standard format used for the API endpoint URL’s

``` r
dateconv<-function(date){
 date<-as.Date(date, tryFormats=c("%m-%d-%y","%m-%d-%Y","%m/%d/%y", "%m/%d/%Y",   "%B %d %Y", "%Y-%m-%d", "%Y/%m/%d", "%B %d, %Y","%b %d, %Y", "%b %d %Y", "%B %d %y", "%B %d, %y", "%b %d %y", "%b %d, %y" ), optional=TRUE)
return(date)
}
```

## *Imagery* Function

This function is used to provide the URL to access the Landsat 8 image
for a specific location and date.

``` r
Imagery<-function(latitude, longitude, date, api_key){
  #convert date to account for multiple formats
  date<-dateconv(date)
#get the data from the API
outputAPI<-paste0("https://api.nasa.gov/planetary/earth/imagery?lon=",longitude, "&lat=",latitude,"&date=",date,"&dim=0.15","&api_key=",api_key)

outputAPI
}
```

## *Assets* Function

This function is used to access the date times and asset names for the
closest available imagery.

``` r
Assets<-function(latitude, longitude, date,api_key){
  #convert date to account for multiple formats
  date<-dateconv(date)
#get the data from the API
outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/planetary/earth/assets?lon=",longitude, "&lat=",latitude,"&date=",date,"&dim=0.15","&api_key=",api_key)))
as.tibble(outputAPI)
}
```

## *Neo-Feed* Function

This function was created to gather a list of asteroids between two
dates and lists them based on their closest approach to earth.

``` r
NeoFeed<-function(start_date, end_date, api_key){
 start_date<-dateconv(start_date)
 end_date<-dateconv(end_date)
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/neo/rest/v1/feed?start_date=",start_date,"&end_date=",end_date,"&api_key=",api_key)))
 as.tibble(ouputAPI)
}
```

## *Neo-Lookup* Function

This function was created to allow the user to look up an asteroid based
on its NASA ID.

``` r
NeoLookup<-function(asteroid_id, api_key){
 outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/neo/rest/v1/neo/",asteroid_id,"?api_key=",api_key)))
 as.tibble(ouputAPI)
}
```

## *Weather* Function

This function allows the user to look up certain space weather events
based on date and type of event.

``` r
Weather<-function(start_date,end_date,event,api_key){
  
start_date<-dateconv(start_date)
end_date<-dateconv(end_date)

#allow for multiple event entries to make the function more user friendly.  Must convert event entry to all lower case letters  first to account for varying capitalization.  

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


outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/DONKI/",event,"?startDate=",start_date,"&endDate=",end_date,"&api_key=",api_key)))
as.tibble(outputAPI)
}
```

## Techport Function

This function allows the user to make queries about NASA’s technology
development programs.

``` r
Techport<-function(parameter_id, api_key){
  outputAPI<-fromJSON(getURL(paste0("https://api.nasa.gov/techport/api/projects/",id_parameter,"?api_key=",api_key)))
as.tibble(outputAPI)
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
else if (func == "Neo-Feed"){
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

\#Exploratory Data Analysis

Now that we have our functions, we can do some data exploration.

I am going to start with the Weather function, and pull data from solar
flares that occured in
2019.

``` r
#use the NASAAPI function to pull solar flare events in 2019 and save the output as an object.
SF<-NASAAPI("Weather", "Jan 1 2019", "Dec 1 2019", "solar flare","zMTdCaPaYeIjYgd9N91EsaFUvxYsCMR1o32ih13X")
```

    ## Warning: `as.tibble()` was deprecated in tibble 2.0.0.
    ## Please use `as_tibble()` instead.
    ## The signature and semantics have changed, see `?as_tibble`.

``` r
SF
```

    ## # A tibble: 7 x 10
    ##   flrID   instruments  beginTime  peakTime endTime classType sourceLocation activeRegionNum linkedEvents link     
    ##   <chr>   <list>       <chr>      <chr>    <lgl>   <chr>     <chr>                    <int> <list>       <chr>    
    ## 1 2019-0… <df [1 × 1]> 2019-01-2… 2019-01… NA      C5.0      N05W26                   12733 <NULL>       https://…
    ## 2 2019-0… <df [1 × 1]> 2019-01-3… 2019-01… NA      C5.2      N05W79                   12733 <NULL>       https://…
    ## 3 2019-0… <df [1 × 1]> 2019-03-0… 2019-03… NA      C1.3      N09W04                   12734 <df [1 × 1]> https://…
    ## 4 2019-0… <df [1 × 1]> 2019-03-2… 2019-03… NA      B6.1      N09W23                   12736 <df [1 × 1]> https://…
    ## 5 2019-0… <df [1 × 1]> 2019-03-2… 2019-03… NA      C4.8      N08W26                   12736 <df [1 × 1]> https://…
    ## 6 2019-0… <df [1 × 1]> 2019-03-2… 2019-03… NA      C5.6      N08W34                   12736 <NULL>       https://…
    ## 7 2019-0… <df [1 × 1]> 2019-05-0… 2019-05… NA      C9.9      N08E50                   12740 <NULL>       https://…

For the purposes of this analysis, I want to modify the character string
containing the start times and peak times. First I will remove the “T”
and “Z” so only the date and time are returned, then I will convert
those into chron values that can be used in
calculations.

``` r
#remove the additional information from the start times for CME's and Solar Flares and convert the string into date format including hours and minutes
 a<-substr(SF$peakTime, 1, 10) #substring containing the date only of peak
 b<-substr(SF$peakTime, 12, 16) #substring containing the hours and minutes of peak
 c<-paste(a,b) #combine substrings into one character string 
 d<-substr(SF$beginTime, 1, 10) #substring containing the date only of begin
 e<-substr(SF$beginTime, 12, 16) #substring containing the hours and minutes of begin
 f<-paste(d,e) #combine substrings into one character string

 #convert both strings to chron format and replace existing variables
SF$beginTime<-as.chron(c, format="%Y-%m-%d  %H:%M") 
SF$peakTime<-as.chron(f, format="%Y-%m-%d  %H:%M")
```

Now that the data has been converted into a chron format, I can do
calculations with the dates. I want to start by creating a variable in
the solar flare data set that measures the time difference between the
beginning time and the peak of each solar flare event (in seconds).

``` r
SF<-SF %>% mutate(begin_peak=difftime(beginTime,peakTime,units="secs"))
```

Now that I have the difference time, I want to create a categorical
variable that separates the begin\_peak variable into groups by length
of time difference: short (\<1000 seconds), medium (1000 to 2000
seconds), and long(\>2000
seconds).

``` r
SF<-SF %>% mutate(length=if_else(begin_peak>2000, "Long", if_else(begin_peak >=1000, "Medium","Short")))

SF
```

    ## # A tibble: 7 x 12
    ##   flrID   instruments  beginTime  peakTime endTime classType sourceLocation activeRegionNum linkedEvents link     
    ##   <chr>   <list>       <chron>    <chron>  <lgl>   <chr>     <chr>                    <int> <list>       <chr>    
    ## 1 2019-0… <df [1 × 1]> (01/26/19… (01/26/… NA      C5.0      N05W26                   12733 <NULL>       https://…
    ## 2 2019-0… <df [1 × 1]> (01/30/19… (01/30/… NA      C5.2      N05W79                   12733 <NULL>       https://…
    ## 3 2019-0… <df [1 × 1]> (03/08/19… (03/08/… NA      C1.3      N09W04                   12734 <df [1 × 1]> https://…
    ## 4 2019-0… <df [1 × 1]> (03/20/19… (03/20/… NA      B6.1      N09W23                   12736 <df [1 × 1]> https://…
    ## 5 2019-0… <df [1 × 1]> (03/20/19… (03/20/… NA      C4.8      N08W26                   12736 <df [1 × 1]> https://…
    ## 6 2019-0… <df [1 × 1]> (03/21/19… (03/21/… NA      C5.6      N08W34                   12736 <NULL>       https://…
    ## 7 2019-0… <df [1 × 1]> (05/06/19… (05/06/… NA      C9.9      N08E50                   12740 <NULL>       https://…
    ## # … with 2 more variables: begin_peak <drtn>, length <chr>

With this final data frame, I can create a contingency table of the
length and the Active Region Number of the flare.

``` r
table(SF$activeRegionNum,SF$length)
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
bar<-ggplot(SF, aes(x=as.character(activeRegionNum)))
bar+geom_bar(aes(fill=as.factor(length)), position="dodge")+labs(x="Active Region Number", y="Count", title="Bar Plot of Solar Flare Events per Region by Length")+scale_fill_discrete(name="Length")
```

![](README_files/figure-gfm/barplot1-1.png)<!-- -->
