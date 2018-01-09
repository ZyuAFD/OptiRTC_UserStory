## Loading Sample Data for Aggregation Process


### packages required for analysis
```
library(tidyverse)
library(data.table)
library(knitr)
library(kableExtra)
library(magrittr)
library(lubridate)
library(units)
```

### Loading sample data
The sample data used for demonstrating the methods in this book is monitored at a green roof located in UWM Golda Meir library in Milwaukee, WI. It includes meastures:

* **time** (yyyy-mm-dd HH:MM:SS) in UTC time zone
* **instant rain intensity** in inches for each time step
* **temperature** in F 
* **estimated soil moisture** in water volume content from four sensors in different locations 
* **overall estimated soil moisture** in water volume content for the whole site

```

filepath='https://github.com/ZyuAFD/OptiRTC_UserStory/raw/master/Sample%20Data/gold-meir-2017-feb-download.csv'

GoldaMeir_Dt=fread(filepath,
                           col.names=c('Time',
                                       'Inst_Rain',
                                       'Temp',
                                       'Est_North_SoilM',
                                       'Est_NorthCtr_SoilM',
                                       'Est_SouthCtr_SoilM',
                                       'Est_South_SoilM',
                                       'Est_SoilM')) %>% 
    # Convert time information from character to time type using "lubridate" package
    mutate(Time=ymd_hms(Time,tz="UTC")) %>% 
    #Assign unit information to each data stream
    mutate(Inst_Rain=Inst_Rain * make_unit("inch"),
            Temp=Temp * make_unit("degF"),
            Est_North_SoilM=Est_North_SoilM * make_unit("%"),
            Est_NorthCtr_SoilM=Est_NorthCtr_SoilM * make_unit("%"),
            Est_SouthCtr_SoilM=Est_SouthCtr_SoilM * make_unit("%"),
            Est_South_SoilM=Est_South_SoilM * make_unit("%"),
            Est_SoilM=Est_SoilM * make_unit("%")) 
    
GoldaMeir_Dt %>% head %>% kable

```