ST558 Project 1
================
Jenna Wilkie
10/2/2021

## Requirements

I used the following packages to interact with the NASA API

*tidyverse*: A package useful for data manipulation. Installation of the
*tidyverse* package also includes the *readr*, *tidyr*, *dplyr*, and
*tibble* pakages.  
*jsonlite*: A function used to connect to the API through URL *magick*:
library used to save image from API URL *httr*: used to connect to URLs

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

This function is used to access the Landsat 8 image for a specific
location and date.

``` r
Imagery<-function(latitude, longitude, date,api_key){
  #convert date to account for multiple formats
  date<-dateconv(date)
#get the data from the API
outputAPI<-paste0("https://api.nasa.gov/planetary/earth/imagery?lon=",longitude, "&lat=",latitude,"&date=",date,"&dim=0.15","&api_key=",api_key)

#save the returned image as an object
output_image<-image_read(outputAPI)
print(output_image)
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
outputAPI<-jsonlite::fromJSON((paste0("https://api.nasa.gov/planetary/earth/assets?lon=",longitude, "&lat=",latitude,"&date=",date,"&dim=0.15","&api_key=",api_key)))
tbl_df(outputAPI)
}
```

\#Neo-Feed Function

This function was created to gather a list of asteroids between two
dates and lists them based on their closest approach to earth.

``` r
NeoFeed<-function(start_date, end_date, api_key){
 date<-convdate(date)
 outputAPI<-jsonlite::fromJSON(paste0("https://api.nasa.gov/neo/rest/v1/feed?start_date=",start_date,"&end_date=",end_date,"&api_key=",api_key))
 tbl_df(ouputAPI)
}
```

\#Neo-Lookup Function

This function was created to allow the user to look up an asteroid based
on its NASA ID.

``` r
NeoLookup<-function(asteroid_id, api_key){
 date<-convdate(date)
 outputAPI<-jsonlite::fromJSON(paste0("https://api.nasa.gov/neo/rest/v1/neo/",asteroid_id,"?api_key=",api_key))
 tbl_df(ouputAPI)
}
```

\#Weather Events Function

This function allows the user to look up certain space weather events
based on date and type of event.

``` r
Weather<-function(start_date,end_date,event,api_key){
#allow for multiple event entries to make the function more user friendly.  Must convert event to all lower case first to account for varying capitalization.  
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
else if (event=="radiation belt enhancement"||"rbe"||"radiation belt enhancement (rbe)"){
  event<-"RBE"
}
else if (event=="hight speed stream"||event=="hss"||event=="hight speed stream (hss)"){
  event<-"HSS"
}
else {warning("Warning: Invalid weather event entry.")}

outputAPI<-jsonlite::fromJSON(paste0("https://api.nasa.gov/DONKI/",event,"?startDate=",start_date,"&endDate=",end_date,"&api_key=",api_key))
tbl_df(outputAPI)
}
```

# Techport Function

This function allows the user to make queries about NASA’s technology
development programs.

``` r
techport<-function(parameter_id, api_key){
  outputAPI<-jsonlite::JSON(paste0("https://api.nasa.gov/techport/api/projects/",id_parameter,"?api_key=",api_key))
tbl_df(outputAPI)
}
```
