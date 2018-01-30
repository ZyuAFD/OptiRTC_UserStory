
## Average Approximating on Continuous Information

Agregation process on discete information will be described here using temperature data. 5 minutes time interval will be used for curating the timestamps.

#### Loading Sample Data


```R
library(devtools)
source_url("https://raw.githubusercontent.com/OptiRTC/export-analysis/ZyuAFD-patch-1/Data/Loading%20Sample%20Data.R?token=AKLn5qtCQw2wdWyFj7-K_iBfF_SC7qI1ks5acIeYwA%3D%3D")
```

#### This code is a sample of aggregation of rain gauge data on Sample data. 

- The data quality is first checked of the duplicated time step for Temperature data



```R
# Extract Rain information for analysis
Temp_dt=GoldaMeir_Dt %>% select(Time,Temp) 

Temp_dt %>% head(10)%>% kable

# Check duplicate time step

Temp_dt %>% 
    group_by(Time) %>% 
    tally %>% 
    filter(n>1) %>% 
    arrange(-n)
```


    
    
    |Time                |       Temp|
    |:-------------------|----------:|
    |2015-12-03 15:34:15 |    NA degF|
    |2015-12-03 15:34:01 | 34.71 degF|
    |2015-12-03 15:32:38 |    NA degF|
    |2015-12-03 15:32:34 |    NA degF|
    |2015-12-03 15:30:54 |    NA degF|
    |2015-12-03 15:29:15 |    NA degF|
    |2015-12-03 15:29:01 | 34.71 degF|
    |2015-12-03 15:27:38 |    NA degF|
    |2015-12-03 15:27:34 |    NA degF|
    |2015-12-03 15:25:54 |    NA degF|



<table>
<thead><tr><th scope=col>Time</th><th scope=col>n</th></tr></thead>
<tbody>
</tbody>
</table>



- Time steps are then rounded up to a time interval value of 5 minutes ( This string has to be among the built-in strings for "unit" parameter in [lubridate::round_date](https://github.com/tidyverse/lubridate/blob/master/R/round.r) function). In this case, [lubridate::floordate](https://github.com/tidyverse/lubridate/blob/master/R/round.r) is used since rain gauge data is a cumulative value aggregated down to its nearest round time point.
- Check the duplicated time interval and aggregate the rain data upon it


```R
Time_Intv= "5 mins" # constant of time interval

library(padr)
Temp_dt %>% 
        select(Time) %>% 
    # Get the range of date
        summarize(min(Time),max(Time)) %>% 
    gather() %>% 
    select(value) %>% 
    mutate(value=lubridate::round_date(value,Time_Intv)) %>% 
    # Get the complete time stamps
    pad(interval=Time_Intv) ->Time_cmplt


# Approximate Temperature to target time stamps by linear interpolation 
library(zoo)
Temp_dt_Cur=Time_cmplt %>% 
    mutate(Temp=na.approx(as.numeric(Temp_dt$Temp),x=Temp_dt$Time,xout=Time_cmplt$value,na.rm=F)) %>% 
    mutate(Temp=Temp* make_unit("degF")) %>% 
    rename(Time=value)
      
Temp_dt %>% arrange(Time) %>% head(15)
Temp_dt_Cur %>% arrange(Time) %>% head(15)
```

    
    Attaching package: 'zoo'
    
    The following objects are masked from 'package:base':
    
        as.Date, as.Date.numeric
    
    


<table>
<thead><tr><th scope=col>Time</th><th scope=col>Temp</th></tr></thead>
<tbody>
	<tr><td>2015-08-21 19:34:01</td><td>71.100 degF        </td></tr>
	<tr><td>2015-08-21 19:37:38</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:39:01</td><td>70.585 degF        </td></tr>
	<tr><td>2015-08-21 19:39:15</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:40:54</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:42:34</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:42:38</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:44:01</td><td>70.070 degF        </td></tr>
	<tr><td>2015-08-21 19:44:15</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:45:54</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:47:34</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:47:38</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:49:01</td><td>70.305 degF        </td></tr>
	<tr><td>2015-08-21 19:49:15</td><td>    NA degF        </td></tr>
	<tr><td>2015-08-21 19:50:54</td><td>    NA degF        </td></tr>
</tbody>
</table>




<table>
<thead><tr><th scope=col>Time</th><th scope=col>Temp</th></tr></thead>
<tbody>
	<tr><td>2015-08-21 19:35:00</td><td>70.99872 degF      </td></tr>
	<tr><td>2015-08-21 19:40:00</td><td>70.48372 degF      </td></tr>
	<tr><td>2015-08-21 19:45:00</td><td>70.11622 degF      </td></tr>
	<tr><td>2015-08-21 19:50:00</td><td>70.25878 degF      </td></tr>
	<tr><td>2015-08-21 19:55:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:00:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:05:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:10:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:15:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:20:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:25:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:30:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:35:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:40:00</td><td>70.07000 degF      </td></tr>
	<tr><td>2015-08-21 20:45:00</td><td>70.05722 degF      </td></tr>
</tbody>
</table>



Since the timestamps have already curated to complete series (no gaps), there is no need to check the data gaps in the result dataset.
