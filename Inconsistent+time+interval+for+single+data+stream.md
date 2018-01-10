
## Story 1: Inconsistent time intervals for individual data streams

The OptiRTC data is collect on real time. The inconsistent interval of the logged data makes it difficult to import the data into some software application or analysis software without significant manipulating and reanalysis. Code is needed that normalizes the time interval of the data streams to user-specified values. 

#### Loading Sample Data


```R
# code from https://tonybreyal.wordpress.com/2011/11/24/source_https-sourcing-an-r-script-from-github/
# for loading github code without SSL verification
source_https <- function(url, ...) {
  # load package
  require(RCurl)
 
  # parse and evaluate each .R script
  sapply(c(url, ...), function(u) {
    eval(parse(text = getURL(u, followlocation = TRUE, cainfo = system.file("CurlSSL", "cacert.pem", package = "RCurl"))), envir = .GlobalEnv)
  })
}
source_https("https://raw.githubusercontent.com/OptiRTC/export-analysis/ZyuAFD-patch-1/Data/Loading%20Sample%20Data.R?token=AKLn5hDhdHsZqC5eLuQoMFs5obp-79keks5aX59SwA%3D%3D")
```

#### This code is a sample of aggregation of rain gauge data on Golda Meir data. 

- The data quality is first checked of the duplicated time step for rain data


```R

# Extract Rain information for analysis
Rain_dt=GoldaMeir_Dt %>% select(Time,Inst_Rain) %>% filter(!is.na(Inst_Rain))

Rain_dt %>% head(10)%>% kable

# Check duplicate time step

Rain_dt %>% 
    group_by(Time) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n) %>% 
    kable
```


    
    
    |Time                | Inst_Rain|
    |:-------------------|---------:|
    |2015-12-03 15:34:01 |    0 inch|
    |2015-12-03 15:29:01 |    0 inch|
    |2015-12-03 15:24:01 |    0 inch|
    |2015-12-03 15:19:01 |    0 inch|
    |2015-12-03 15:14:01 |    0 inch|
    |2015-12-03 15:09:01 |    0 inch|
    |2015-12-03 15:04:01 |    0 inch|
    |2015-12-03 14:59:01 |    0 inch|
    |2015-12-03 14:54:01 |    0 inch|
    |2015-12-03 14:49:01 |    0 inch|



    
    
    |Time |  n|
    |:----|--:|


- Time steps are then rounded up to a time interval value of 5 minutes ( This string has to be among the built-in strings for "unit" parameter in [lubridate::round_date](https://github.com/tidyverse/lubridate/blob/master/R/round.r) function). In this case, [lubridate::floordate](https://github.com/tidyverse/lubridate/blob/master/R/round.r) is used since rain gauge data is a cumulative value aggregated down to its nearest round time point.
- Check the duplicated time interval and aggregate the rain data upon it


```R

Time_Intv= "hour" # constant of time interval in lubridate::round_date function

# Round time into specified time interval
Rain_dt %<>% 
    mutate(Time_rnd=lubridate::floor_date(Time,Time_Intv))

Rain_dt %>% head%>% kable

# Check duplicate time on rounded time 
Rain_dt %>% 
    group_by(Time_rnd) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n)  %>% 
    kable

# Aggregate the rain amount 
Rain_dt %<>% 
    group_by(Time_rnd) %>% 
    summarise(Inst_Rain=sum(Inst_Rain))

Rain_dt %>% head(10)%>% kable
```


    
    
    |Time                | Inst_Rain|Time_rnd            |
    |:-------------------|---------:|:-------------------|
    |2015-12-03 15:34:01 |    0 inch|2015-12-03 15:00:00 |
    |2015-12-03 15:29:01 |    0 inch|2015-12-03 15:00:00 |
    |2015-12-03 15:24:01 |    0 inch|2015-12-03 15:00:00 |
    |2015-12-03 15:19:01 |    0 inch|2015-12-03 15:00:00 |
    |2015-12-03 15:14:01 |    0 inch|2015-12-03 15:00:00 |
    |2015-12-03 15:09:01 |    0 inch|2015-12-03 15:00:00 |



<table>
<thead><tr><th scope=col>Time_rnd</th><th scope=col>n</th></tr></thead>
<tbody>
	<tr><td>2015-08-21 20:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-21 21:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-21 22:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-21 23:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 00:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 01:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 02:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 03:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 04:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 05:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 06:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 07:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 08:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 09:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 10:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 11:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 12:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 13:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 14:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 15:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 16:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 17:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 18:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 19:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 20:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 21:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 22:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-22 23:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-23 00:00:00</td><td>12                 </td></tr>
	<tr><td>2015-08-23 01:00:00</td><td>12                 </td></tr>
	<tr><td>...</td><td>...</td></tr>
	<tr><td>2015-12-02 11:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 12:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 13:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 14:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 15:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 16:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 17:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 18:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 19:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 20:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 21:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 22:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-02 23:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 00:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 01:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 02:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 03:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 04:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 05:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 06:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 07:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 08:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 09:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 10:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 11:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 12:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 13:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 14:00:00</td><td>12                 </td></tr>
	<tr><td>2015-12-03 15:00:00</td><td> 7                 </td></tr>
	<tr><td>2015-08-21 19:00:00</td><td> 6                 </td></tr>
</tbody>
</table>




    
    
    |Time_rnd            | Inst_Rain|
    |:-------------------|---------:|
    |2015-08-21 19:00:00 |    0 inch|
    |2015-08-21 20:00:00 |    0 inch|
    |2015-08-21 21:00:00 |    0 inch|
    |2015-08-21 22:00:00 |    0 inch|
    |2015-08-21 23:00:00 |    0 inch|
    |2015-08-22 00:00:00 |    0 inch|
    |2015-08-22 01:00:00 |    0 inch|
    |2015-08-22 02:00:00 |    0 inch|
    |2015-08-22 03:00:00 |    0 inch|
    |2015-08-22 04:00:00 |    0 inch|


- Check the data gaps in the aggregated data


```R

FindGaps=function(Dt,Tm_intv)
    #Dt dataframe with time column
    #Tm_intv: min gap, use time interval here
{
    Dt %>% 
        arrange(Time) %>% 
        mutate(lagT=lag(Time)) %>% 
        filter(as.period(interval(lagT,Time))>period(Tm_intv))->gaps
    
    if (nrow(gaps)==0) return("No Gaps")
    
    gaps %>% 
        mutate(Time_interval=interval(lagT,Time)) %>% 
        select(Time_interval) %>% 
        mutate(Period=as.period(Time_interval)) %>% 
        return
}

Rain_dt %>% 
    rename(Time=Time_rnd) %>% 
    FindGaps(Tm_intv=Time_Intv) %>% 
    kable
```


    
    
    |x       |
    |:-------|
    |No Gaps |
