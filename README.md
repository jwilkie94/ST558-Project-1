ST558 Project 1
================
Jenna Wilkie
10/2/2021

## Requirements

I used the following packages to interact with the NASA API

*tidyverse*: A package useful for data manipulation. Installation of the
*tidyverse* package also includes the *readr*, *tidyr*, *dplyr*, and
*tibble* pakages.  
*jsonlite*: A function used to connect to the API through URL *jpeg*:
library used to save image from API URL

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
Imagery<-function(latitude, longitude, date,apikey){
  #convert date to account for multiple formats
  date<-dateconv(date)
#get the data from the API
outputAPI<-paste0("https://api.nasa.gov/planetary/earth/imagery?lon=",longitude, "&lat=",latitude,"&date=",date,"&api_key=",apikey)

#save the returned image as an object
output_image<-readJPEG(fromJSON(outputAPI))
return(output_image)
}
```
