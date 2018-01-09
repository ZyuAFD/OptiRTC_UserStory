## Story 1: Inconsistent time intervals for individual data streams

The OptiRTC data is collect on real time. The inconsistent interval of the logged data makes it difficult to import the data into some software application or analysis software without significant manipulating and reanalysis. Code is needed that normalizes the time interval of the data streams to user-specified values. 


### packages required for analysis
```
library(tidyverse)
library(data.table)
library(RcppRoll)
library(knitr)
library(kableExtra)
```

### Loading sample data
```{r}

filepath='https://github.com/ZyuAFD/OptiRTC_UserStory/raw/master/Sample%20Data/gold-meir-2017-feb-download.csv'

GoldaMeir_Dt_Feb2017=fread(filepath,
                           col.names=c('Time_UTC',
                                       'Inst_Rain_in',
                                       'Temp_F',
                                       'Est_North_SoilM',
                                       'Est_NorthCtr_SoilM',
                                       'Est_SouthCtr_SoilM',
                                       'Est_South_SoilM',
                                       'Est_SoilM')) %>% 
    mutate(Time=ymd_hms(Time_UTC),
           Time_Stamp=ymd_hm(substr(Time_UTC,1,16)))

rm(path,file)
```


### This code is a sample of aggregation of rain gauge data on Golda Meir data. 

- The time interval could be specified by usersh. This value has to be the built-in strings for "unit" parameter in [lubridate::round_date](https://github.com/tidyverse/lubridate/blob/master/R/round.r) function.
- The data quality is first checked of the duplicated time step for rain data

```
#Time_Intv= "hour" # constant of time interval in lubridate::round_date function

GoldaMeir_Dt_Feb2017 %>% head(10) %>% kable

# Extract Rain information for analysis
Rain_dt=GoldaMeir_Dt_Feb2017 %>% select(Time,Inst_Rain_in) %>% filter(!is.na(Inst_Rain_in))

Rain_dt %>% head(10)%>% kable

# Check duplicate time step

Rain_dt %>% 
    group_by(Time) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n)
```

- Time steps are then rounded up to the ceiling time intervals specified by users
- Check the duplicated time interval and aggregate the rain data upon it

```{r Time step round,echo=TRUE}
# Round time into specified time interval
Rain_dt %<>% 
    mutate(Time_rnd=lubridate::ceiling_date(Time,Time_Intv))

Rain_dt %>% head%>% kable

# Check duplicate time on rounded time 
Rain_dt %>% 
    group_by(Time_rnd) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n) 

# Aggregate the rain amount 
Rain_dt %<>% 
    group_by(Time_rnd) %>% 
    summarise(Inst_Rain_in=sum(Inst_Rain_in))

Rain_dt %>% head(10)%>% kable

```

- Check the data gaps in the aggregated data
```{r Gap Check,echo=TRUE}

FindGaps=function(Dt,Tm_intv=Time_Intv)
    #Dt dataframe with time column
    #Tm_intv: min gap, use time interval here
{
    Dt %>% 
        arrange(Time) %>% 
        mutate(lagT=lag(Time)) %>% 
        filter(as.period(interval(lagT,Time))>period(Time_Intv))->gaps
    
    
    gaps %>% 
        mutate(Intv=interval(lagT,Time)) %>% 
        select(Intv) %>% 
        mutate(Prd=as.period(Intv)) %>% 
        return
}

Rain_dt %>% 
    rename(Time=Time_rnd) %>% 
    FindGaps %>% 
    rename(Time_interval=Intv, Period=Prd) %>% 
    kable

```






