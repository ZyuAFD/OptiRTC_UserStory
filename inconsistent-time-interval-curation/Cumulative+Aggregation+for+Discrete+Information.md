
## Cumulative Aggregation for Discrete Information

Agregation process on discete information will be described here using rain data. 5 minutes time interval will be used for curating the timestamps.  

#### Loading Sample Data


```R
library(devtools)
source_url("https://raw.githubusercontent.com/OptiRTC/export-analysis/ZyuAFD-patch-1/Data/Loading%20Sample%20Data.R?token=AKLn5qtCQw2wdWyFj7-K_iBfF_SC7qI1ks5acIeYwA%3D%3D")
```

#### This code is a sample of aggregation of rain gauge data on sample data. 

- The data quality is first checked of the duplicated time step for rain data


```R

# Extract Rain information for analysis
Rain_dt=GoldaMeir_Dt %>% select(Time,Inst_Rain) %>% filter(!is.na(Inst_Rain))

Rain_dt %>% head(10)

# Check duplicate time step

Rain_dt %>% 
    group_by(Time) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n)
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

Time_Intv= "5 mins" # constant of time interval in lubridate::round_date function

# Round time into specified time interval
Rain_dt %<>% 
    mutate(Time_rnd=lubridate::floor_date(Time,Time_Intv))

Rain_dt %>% head

# Check duplicate time on rounded time 
Rain_dt %>% 
    group_by(Time_rnd) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n)  %>% 
    head 

# Aggregate the rain amount 
Rain_dt %<>% 
    group_by(Time_rnd) %>% 
    summarise(Inst_Rain=sum(Inst_Rain))

Rain_dt %>% head(10)
```


    
    
    |Time                | Inst_Rain|Time_rnd            |
    |:-------------------|---------:|:-------------------|
    |2015-12-03 15:34:01 |    0 inch|2015-12-03 15:30:00 |
    |2015-12-03 15:29:01 |    0 inch|2015-12-03 15:25:00 |
    |2015-12-03 15:24:01 |    0 inch|2015-12-03 15:20:00 |
    |2015-12-03 15:19:01 |    0 inch|2015-12-03 15:15:00 |
    |2015-12-03 15:14:01 |    0 inch|2015-12-03 15:10:00 |
    |2015-12-03 15:09:01 |    0 inch|2015-12-03 15:05:00 |



    
    
    |Time_rnd |  n|
    |:--------|--:|



    
    
    |Time_rnd            | Inst_Rain|
    |:-------------------|---------:|
    |2015-08-21 19:30:00 |    0 inch|
    |2015-08-21 19:35:00 |    0 inch|
    |2015-08-21 19:40:00 |    0 inch|
    |2015-08-21 19:45:00 |    0 inch|
    |2015-08-21 19:50:00 |    0 inch|
    |2015-08-21 19:55:00 |    0 inch|
    |2015-08-21 20:00:00 |    0 inch|
    |2015-08-21 20:05:00 |    0 inch|
    |2015-08-21 20:10:00 |    0 inch|
    |2015-08-21 20:15:00 |    0 inch|


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
    FindGaps(Tm_intv=Time_Intv) 
```


    
    
    |x       |
    |:-------|
    |No Gaps |


If there have some gaps, the result data should be join to or filled with missing time stamps to obtain a complete data.


```R
# add a gap to data
Rain_dt %<>% 
    filter(!data.table::between(Time_rnd, ymd('2015-09-01'),ymd_hm('2015-09-01 0:20')))
           
Rain_dt %>% 
    rename(Time=Time_rnd) %>% 
    FindGaps(Tm_intv=Time_Intv) 

# make complete time stamp dataset
library(padr)
Rain_dt %<>% pad

Rain_dt %>% 
    filter(data.table::between(Time_rnd, ymd('2015-09-01'),ymd('2015-09-02'))) %>% 
    head(20) %>% kable
```


<table>
<thead><tr><th scope=col>Time_interval</th><th scope=col>Period</th></tr></thead>
<tbody>
	<tr><td>2015-08-31 23:55:00 UTC--2015-09-01 00:25:00 UTC</td><td>30M 0S                                          </td></tr>
</tbody>
</table>



    pad applied on the interval: 5 min
    


    
    
    |Time_rnd            | Inst_Rain|
    |:-------------------|---------:|
    |2015-09-01 00:00:00 |        NA|
    |2015-09-01 00:05:00 |        NA|
    |2015-09-01 00:10:00 |        NA|
    |2015-09-01 00:15:00 |        NA|
    |2015-09-01 00:20:00 |        NA|
    |2015-09-01 00:25:00 |         0|
    |2015-09-01 00:30:00 |         0|
    |2015-09-01 00:35:00 |         0|
    |2015-09-01 00:40:00 |         0|
    |2015-09-01 00:45:00 |         0|
    |2015-09-01 00:50:00 |         0|
    |2015-09-01 00:55:00 |         0|
    |2015-09-01 01:00:00 |         0|
    |2015-09-01 01:05:00 |         0|
    |2015-09-01 01:10:00 |         0|
    |2015-09-01 01:15:00 |         0|
    |2015-09-01 01:20:00 |         0|
    |2015-09-01 01:25:00 |         0|
    |2015-09-01 01:30:00 |         0|
    |2015-09-01 01:35:00 |         0|

