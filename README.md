# OptiRTC User Stories
This is a book to describe the user stories in using OptiRTC data. It provides a coding guidance in manipulating, analysing, modelling for general hydrology purpose using monitoring data collected by OptiRTC inc.The purpose is to improve the adaptability of OptiRTC data output to potential users from consulting, government, and academia. These users, who could include water resource engineers, hydrology and hydraulic modelers, project managers, graduate students, and professors, may not have an extensive background in IT and/or data analytics, and thus currently have difficulty manipulating and/or using data stored on the Opti system for analysis purposes. This proposal would develop a “toolbox” for these users to seamlessly transform data stored on the Opti-system into .csv files that are ready to use in Excel, R, Matlab, and various hydrologic and hydraulic models (especially EPA SWMM).

The stories listed below are all collected from consulting and academia, will be investigated and interpreted by codes in this book: 

1. **Inconsistent time intervals for individual data streams:** The inconsistent interval of the logged data makes it difficult to import the data into some software application or analysis software without significant manipulating and reanalysis. Code is needed that normalizes the time interval of the data streams to user-specified values. 

2. **Inconsistent time intervals across multiple data streams logged at the same site:** Different time steps across different data streams makes it difficult to create one neat data summary file for all data collected at a specific project. Code is needed that synchronizes the time stamp of all data streams from user-specified clusters of sites. 

3. **Difficulties computing totals, averages, unit conversions, or data transformations at user-specified time intervals:** Users who need to adjust the time interval of the data (for example computing hourly or daily totals), cannot do this without first manipulating and reanalyzing the data. Code is needed that would allow users to export and transform the data in highly customized ways. 

4. **Difficulty filtering and interpolating missing or invalid data points:** Where data is missing or invalid, users often seek to interpolate values, after passing the raw data through some quality control filters. Currently, they are required to do this in a separate application. Code is needed that allows users to pass the raw data through quality control filters, interpolating and missing bad data, and reporting the percent of missing data.

5. **Identifying data gaps:** Data gaps may disqualify certain sets of observations in analyses. Currently, data gaps must be identified by inspection. Code is needed that labels all data gaps, before and after any potential filtering, quality control, and interpolation. 


