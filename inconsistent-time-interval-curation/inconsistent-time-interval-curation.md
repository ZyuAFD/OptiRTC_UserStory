### Story 1: Inconsistent Time Interval Curation

---

The OptiRTC data is collect on real time. Although set up on a fixed time interval, the logging time of each measurement may vary from its consistent interval time stamps due to the impact from factors such as environmental conditions, data transferring lag. Then, the concequent inconsistent interval  makes it difficult to import the data into some software application or analysis without significant manipulating and reanalysis. Code is needed for regulating the time interval of the data streams to user-specified values.

This curation process, however, involves not only regulating time intervals, but also the aggregation processes of all other data streams into the curated time stamps. This section will specifically describe the data manipulating steps for two types of data aggregation.

#### Type 1: cumulative aggregating discrete information

Discrete data measurements, such as rainfall, runoff and CSOs etc, are should be aggregated to the curated timestamps by cumulation.

#### Type 2: average approximating continuous information

Other information, such as temperature, relative humidity and pressure etc, are continuous which should be approximated using average based methods \(e.g. interpolation, smooth\).



