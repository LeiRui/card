# Chapter 4 IoTDB Operation Manual

## 4.1 Scenario Description and Sample Data

To make this manual more practical, we will use a specific scenario example in this chapter to illustrate how to operate IoTDB databases at all stages of use. For your convenience, we also provide you with a sample data file in the scenario of this chapter for you to import into the IoTDB system for trial and operation.

### 4.1.1 Scenario Description
A power department needs to monitor the operation of various power plants under its jurisdiction. By collecting real-time monitoring data sent by various types of sensors deployed by various power plants, the power department can monitor the real-time operation of the power plants and understand the trend of data changes, etc. IoTDB has the characteristics of high write throughput and rich query functions, which can provide effective support for the needs of the power department.

The real-time data needed to be monitored involves multiple attribute layers:

**Power Generation Group**: The data belongs to nearly ten power generation groups, and the name codes are ln, sgcc, etc.

**Power plant**: The power generation group has more than 10 kinds of electric fields, such as wind farm, hydropower plant and photovoltaic power plant, numbered as wf01, wf02, wf03 and so on.

**Device**: Each power plant has about 5,000 kinds of power generation devices such as wind turbines and photovoltaic panels, numbered as wt01, wt02 and so on.

**Sensors**: For different devices, there are 10 to 1000 sensors monitoring different states of the devices , such as power supply status sensor (named status), temperature sensor (named temperature), hardware version sensor (named hardware), etc. 

It is worth noting that prior to the use of IoTDB by the power sector, some historical monitoring data of various power plants needs to be imported into the IoTDB system (we will introduce the import method in Section 4.3.1 of this chapter). Simutaneouly, the real-time monitoring data is continuously flowing into the IoTDB system (we will introduce the import method in Section 4.3.2 of this chapter). 

### 4.1.2 Sample Data
Based on the description of the above sample scenarios, we provide you with a simplified sample data. The data download address is http://tsfile.org/download.

The basic information of the data is shown in Table 4-1.
 
<center>** Table 4-1 The basic information of the data **</center>

|Name  |Data Type|  Coding | Meaning |
|:---|:---|:---|:---|
|root.ln.wf01.wt01.status|   Boolean|PLAIN| the power supply status of  ln group wf01 plant wt01 device |
|root.ln.wf01.wt01.temperature  |Float|RLE| the temperature of ln group wf01 plant wt01 device|
|root.ln.wf02.wt02.hardware  |Text|PLAIN| the hardware version of ln group wf02 plant wt02 device|
|root.ln.wf02.wt02.status  |Boolean|PLAIN| the power supply status of  ln group wf02 plant wt02 device|
|root.sgcc.wf03.wt01.status|Boolean|PLAIN| the power supply status of  sgcc group wf03 plant wt01 device|
|root.sgcc.wf03.wt01.temperature   |Float|RLE| the temperature of sgcc group wf03 plant wt01 device|

The time span of this data is from 10:00 on November 1, 2017 to 12:00 on November 2, 2017. The frequency at which data is generated is two minutes each.

In Section 4.2, we will show how to apply IoTDB's data model rules to construct the data model shown in Section 4.1.2 using  the data from the scenario in Section 4.1.1. In section 4.3.1, we will introduce you to the method of importing historical data, and in section 4.3.2, we will introduce you to the method of accessing real-time data. In section 4.4, we will introduce you to three typical data query patterns using IoTDB. In Section 4.5, we will show you how to update and delete data using IoTDB.

## 4.2 Data Model Selection and Creation
Before importing data to IoTDB, we first select the appropriate data storage model according to the sample data provided in Section 4.1, and then create the storage group and time series using SET STORAGE GROUP statement and CRATE TIMESERIES statement respectively (see Section 7.1.2.2 of this manual for detailed grammar).

### 4.2.1 Storage Model Selection
According to the data attribute layers described in Section 4.1.1 of this chapter, we can express it as an attribute hierarchy structure based on the coverage of attributes and the subordinate relationship between them, as shown in Figure 4.1 below. Its hierarchical relationship is: group layer - electric field layer - device layer - sensor layer. ROOT is the root node, and each node of sensor layer is called a leaf node. In the process of using IoTDB, you can directly connect the attributes on the path from ROOT node to each leaf node with ".", thus forming the name of a time series in IoTDB. For example, The left-most path in Figure 4.1 can generate a time series named `ROOT.ln.wf01.wt01.status`.

<center>![](./chapter4-fig/4.1.jpg)</center>
<center>**Figure 4.1 Attribute hierarchy structure**</center>

After getting the name of the time series, we need to set up the storage group according to the actual scenario and scale of the data. Because in the scenario of this chapter data is usually arrived in the unit of groups (i.e., data may be across electric fields and devices), in order to avoid frequent switching of IO when writing data, and to meet the user's requirement of physical isolation of data in the unit of  groups, we set the storage group at the group layer.

### 4.2.2 Storage Group Creation
After selecting the storage model, according to which we can set up the corresponding storage group. The SQL statements for creating storage groups are as follows:

```
IoTDB > set storage group to root.ln
IoTDB > set storage group to root.sgcc
```

We can thus create two storage groups using the above two SQL statements.

It is worth noting that when the path itself or the parent/child layer of the path is already set as a storage group, the path is then not allowed to be set as a storage group. For example, it is not feasible to set `root.ln.wf01` as a storage group when there exist two storage groups `root.ln` and `root.sgcc`. The system will give the corresponding error message as shown below:

```
IoTDB> set storage group to root.ln.wf01
error: The prefix of root.ln.wf01 has been set to the storage group.
```

### 4.2.3 Show Storage Group
After the storage group is created, we can use the SHOW STORAGE GROUP statement to view all the storage groups. The SQL statement is as follows:

```
IoTDB> show storage group
```

The result is as follows:
![](./chapter4-fig/4.2.jpg)

### 4.2.4 Time Series Creation
According to the storage model selected in Section 4.2.1, we can create corresponding time series in the two storage groups respectively. The SQL statements for creating time series are as follows:

```
IoTDB > create timeseries root.ln.wf01.wt01.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.ln.wf01.wt01.temperature with datatype=FLOAT,encoding=RLE
IoTDB > create timeseries root.ln.wf02.wt02.hardware with datatype=TEXT,encoding=PLAIN
IoTDB > create timeseries root.ln.wf02.wt02.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.sgcc.wf03.wt01.status with datatype=BOOLEAN,encoding=PLAIN
IoTDB > create timeseries root.sgcc.wf03.wt01.temperature with datatype=FLOAT,encoding=RLE
```

It is worth noting that when in the CRATE TIMESERIES statement the encoding method conflicts with the data type, the system will give the corresponding error message as shown below:
```
IoTDB> create timeseries root.ln.wf02.wt02.status WITH DATATYPE=BOOLEAN, ENCODING=TS_2DIFF
error: encoding TS_2DIFF does not support BOOLEAN
```

Please refer to Section 3.3.5 of this manual for correspondence between data type and coding.

### 4.2.5 Show Time Series
Currently, IoTDB supports two ways of viewing time series:

* SHOW TIMESERIES statement presents all time series information in JSON form 
* SHOW TIMESERIES <`Path`> statement returns all time series information and the total number of time series under the given <`Path`>  in tabular form. Time series information includes: timeseries path, storage group it belongs to, data type, coding type.  <`Path`> needs to be a prefix path or a path with star or a timeseries path. SQL statements are as follows:

```
IoTDB> show timeseries root
IoTDB> show timeseries root.ln
```

The results are shown below respectly:
![](./chapter4-fig/4.3.jpg)
![](./chapter4-fig/4.4.jpg)

It is worth noting that when the path queries does not exist, the system will give the corresponding error message as shown below:
```
IoTDB> show timeseries root.ln.wf03
Msg: Failed to fetch timeseries root.ln.wf03's metadata because: Timeseries does not exist.
```

### 4.2.6 Precautions
Version 0.7.0 imposes some limitations on the scale of data that users can operate:

Limit 1: Assuming that the JVM memory allocated to IoTDB at runtime is p and the user-defined size of data in memory written to disk (see `group_size_in_byte` in section 5.2.2.1) is Q, then the number of storage groups should not exceed p/q.

Limit 2: The number of time series should not exceed the ratio of JVM memory allocated to IoTDB at run time to 20KB.

## 4.3 Data Access
### 4.3.1 Import Historical Data
This feature is not supported in version 0.7.0.

### 4.3.2 Import Real-time Data
IoTDB provides users with a variety of ways to insert real-time data, such as directly inputting INSERT statements in Cli/Shell tools (see Section 6.1 of this manual for details), or using Java API (see Section 7.2 for standard JDBC interface) to perform single or batch execution of INSERT statements.

This section mainly introduces the use of INSERT SQL statements for real-time data access in the scenario of Section 4.1.1. See Section 7.1.3.1 for a detailed syntax of INSERT SQL statements.

### 4.3.2.1 Use of INSERT Statements
The INSERT statement can be used to insert data into one or more specified time series that have been created. For each point of data inserted, it consists of a timestamp (see Section 3.1.8) and a sensor acquisition value of a numerical type (see Section 3.2).

In the scenario of this section, take two time series `root.ln.wf02.wt02.status` and `root.ln.wf02.wt02.hardware` as an example, and their data types are BOOLEAN and TEXT, respectively.

The sample code for single column data insertion is as follows:
```
IoTDB > insert into root.ln.wf02.wt02(timestamp,status) values(1,true)
IoTDB > insert into root.ln.wf02.wt02(timestamp,hardware) values(1, "v1")
```

The above example code inserts the long integer timestamp and the value "true" into the time series `root.ln.wf02.wt02.status` and inserts the long integer timestamp and the value "v1" into the time series `root.ln.wf02.wt02.hardware`. When the execution is successful, a prompt "execute successfully" will appear to indicate that the data insertion has been completed.

Note: In IoTDB, TEXT type data can be represented by single and double quotation marks. The insertion statement above uses double quotation marks for TEXT type data. The following example will use single quotation marks for TEXT type data.

The INSERT statement can also support the insertion of multi-column data at the same time point.  The sample code of  inserting the values of the two time series at the same time point '2' is as follows:
```
IoTDB > insert into root.ln.wf02.wt02(timestamp, status, hardware) VALUES (2, false, 'v2')
```

After inserting the data, we can simply query the inserted data using the SELECT statement:
```
IoTDB > select * from root.ln.wf02 where time < 3
```

The result is shown below. From the query results, it can be seen that the insertion statements of single column and multi column data are performed correctly.
![](./chapter4-fig/4.5.jpg)

### 4.3.2.2 Error Handling of INSERT Statements
If the user inserts data into a non-existent time series, for example, execute the following commands:
```
IoTDB > insert into root.ln.wf02.wt02(timestamp, temperature) values(1,"v1")
```
Because `root.ln.wf02.wt02. temperature` does not exist, the system will return the following ERROR information:
```
error: Timeseries root.ln.wf02.wt02.temperature does not exist.
```
If the data type inserted by the user is inconsistent with the corresponding data type of the timeseries, for example, execute the following command:
```
IoTDB > insert into root.ln.wf02.wt02(timestamp,hardware) values(1,100)
```
The system will return the following ERROR information:
```
error: The TEXT data type should be covered by " or '
```

## 4.4 Data Query
### 4.4.1 Time Slice Query
This chapter mainly introduces the relevant examples of time slice query using IoTDB SELECT statements. Detailed SQL syntax and usage specifications can be found in Section 7.1.3.4 of this manual. You can also use the Java JDBC standard interface to execute related queries, as detailed in Section 7.2.

#### 4.4.1.1 Select a Column of Data Based on a Time Interval
The SQL statement is:
```
select temperature from root.ln.wf01.wt01 where time < 2017-11-01T00:08:00.000
```
which means:

The selected device is ln group wf01 plant wt01 device; the selected time series is the temperature sensor (temperature). The SQL statement requires that all temperature sensor values before the time point of "2017-11-01T00:08:00.000" be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.6.jpg)

#### 4.4.1.2 Select Multiple Columns of Data Based on a Time Interval
The SQL statement is:
```
select 
    status, temperature 
from 
    root.ln.wf01.wt01 
where 
    time > 2017-11-01T00:05:00.000 
    and 
    time < 2017-11-01T00:12:00.000
;
```
which means:

The selected device is ln group wf01 plant wt01 device; the selected time series is "status" and "temperature". The SQL statement requires that the status and temperature sensor values between the time point of "2017-11-01T00:05:00.000" and "2017-11-01T00:12:00.000" be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.7.jpg)

#### 4.4.1.3 Select Multiple Columns of Data for the Same Device According to Multiple Time Intervals
IoTDB supports specifying multiple time interval conditions in a query. Users can combine time interval conditions at will according to their needs. For example, the SQL statement is:
```
select 
    status,temperature 
from 
    root.ln.wf01.wt01 
where 
    (time > 2017-11-01T00:05:00.000 and time < 2017-11-01T00:12:00.000) 
    or 
    (time >= 2017-11-01T16:35:00.000 and time <= 2017-11-01T16:37:00.000)
;
```
which means:

The selected device is ln group wf01 plant wt01 device; the selected time series is "status" and "temperature"; the statement specifies two different time intervals, namely "2017-11-01T00:05:00.000 to 2017-11-01T00:12:00.000" and "2017-11-01T16:35:00.000 to 2017-11-01T16:37:00.000". The SQL statement requires that the values of selected time series satisfying any time interval be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.8.jpg)

#### 4.4.1.4 Choose Multiple Columns of Data for Different Devices According to Multiple Time Intervals
The system supports the selection of data in any column in a query, i.e., the selected columns can come from different devices. For example, the SQL statement is:
```
select 
    wf01.wt01.status,wf02.wt02.hardware 
from 
    root.ln 
where 
    (time > 2017-11-01T00:05:00.000 and time < 2017-11-01T00:12:00.000) 
    or 
    (time >= 2017-11-01T16:35:00.000 and time <= 2017-11-01T16:37:00.000)
;
```
which means:

The selected timeseries are "the power supply status of ln group wf01 plant wt01 device" and "the hardware version of ln group wf02 plant wt02 device"; the statement specifies two different time intervals, namely "2017-11-01T00:05:00.000 to 2017-11-01T00:12:00.000" and "2017-11-01T16:35:00.000 to 2017-11-01T16:37:00.000". The SQL statement requires that the values of selected time series satisfying any time interval be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.9.jpg)

### 4.4.2 Down-Frequency Aggregate Query
This chapter mainly introduces the related examples of down-frequency aggregation query, using the GROUP BY clause of IoTDB SELECT statement, which is used to partition the result set according to the user's given partitioning conditions and aggregate the partitioned result set. IoTDB supports partitioning result sets according to time intervals, and by default results are sorted by time in ascending order. Detailed SQL syntax and usage specifications can be found in Section 7.1 of this manual. You can also use the Java JDBC standard interface to execute related queries, as detailed in Section 7.2.

The GROUP BY statement provides users with three types of specified parameters:

* Parameter 1: Time interval for dividing the time axis
* Parameter 2: Time axis origin position (optional)
* Parameter 3: The display window(s) (one or more) on the time axis

The actual meanings of the three types of parameters are shown in Figure 4.2 below. Among them, the paramter 2 is optional. Next we will give three typical examples of frequency reduction aggregation: parameter 2 specified, parameter 2 not specified, and time filtering conditions specified.

![](./chapter4-fig/4.10.jpg)
<center>** Figure 4.2 The actual meanings of the three types of parameters **</center>

#### 4.4.2.1 Down-Frequency Aggregate Query without Specifying the Time Axis Origin Position
The SQL statement is:
```
select 
    count(status), max_value(temperature) 
from 
    root.ln.wf01.wt01 
group by 
    (1d, [2017-11-01T00:00:00, 2017-11-07T23:00:00])
;
```
which means:

Since the user does not specify the time axis origin position, the GROUP BY statement will by default set the origin at 0 (+0 time zone) on January 1, 1970.

The first parameter of the GROUP BY statement above is the time interval for dividing the time axis. Taking this parameter (1d) as time interval and the default origin as the dividing origin, the time axis is divided into several continuous intervals, which are [0,1d], [1d, 2d], [2d, 3d], etc.

The second parameter of the GROUP BY statement above is the display window paramter, which determines the final display range is [2017-11-01T00:00:00, 2017-11-07T23:00:00].

Then the system will use the time and value filtering condition in the WHERE clause and the second parameter of the GROUP BY statement as the data filtering condition to obtain the data satisfying the filtering condition (which in this case is the data in the range of [2017-11-01T00:00:00, 2017-11-07 T23:00:00]), and map these data to the previously segmented time axis (in this case there are mapped data in every 1-day period from 2017-11-01T00:00:00 to 2017-11-07T23:00:00:00).

Since there is data for each time period in the result range to be displayed, the execution result of the SQL statement is shown below:
![](./chapter4-fig/4.11.jpg)

#### 4.4.2.2 Down-Frequency Aggregate Query Specifying the Time Axis Origin Position
The SQL statement is:
```
select 
    count(status), max_value(temperature) 
from 
    root.ln.wf01.wt01 
group by
    (1d, 2017-11-03 00:00:00, [2017-11-01 00:00:00, 2017-11-07 23:00:00])
;
```
which means:

Since the user specifies the time axis origin position parameter as 2017-11-03 00:00:00, the GROUP BY statement will set the origin at 0 (system default time zone) on November 3, 2017.

The first parameter of the GROUP BY statement above is the time interval for dividing the time axis. Taking this parameter (1d) as time interval and the speicified origin as the dividing origin, the time axis is divided into several continuous intervals, which are [2017-11-02T00:00:00, 2017-11-03T00:00:00], [2017-11-03T00:00:00, 2017-11-04T00:00:00], etc.

The third parameter of the GROUP BY statement above is the display window paramter, which determines the final display range is [2017-11-01T00:00:00, 2017-11-07T23:00:00].

hen the system will use the time and value filtering condition in the WHERE clause and the second parameter of the GROUP BY statement as the data filtering condition to obtain the data satisfying the filtering condition (which in this case is the data in the range of [2017-11-01T00:00:00, 2017-11-07T23:00:00]), and map these data to the previously segmented time axis (in this case there are mapped data in every 1-day period from 2017-11-01T00:00:00 to 2017-11-07T23:00:00:00).

Since there is data for each time period in the result range to be displayed, the execution result of the SQL statement is shown below:
![](./chapter4-fig/4.12.jpg)

#### 4.4.2.3 Down-Frequency Aggregate Query Specifying the Time Filtering Conditions
The SQL statement is:
```
select 
    count(status), max_value(temperature) 
from 
    root.ln.wf01.wt01 
where 
    time > 2017-11-03T06:00:00 and temperature > 20 
group by
    (1h, [2017-11-03T00:00:00, 2017-11-03T23:00:00])
;
```
which means:

Since the user does not specify the time axis origin position, the GROUP BY statement will by default set the origin at 0 (+0 time zone) on January 1, 1970.

The first parameter of the GROUP BY statement above is the time interval for dividing the time axis. Taking this parameter (1d) as time interval and the default origin as the dividing origin, the time axis is divided into several continuous intervals, which are [0,1d], [1d, 2d], [2d, 3d], etc.

The second parameter of the GROUP BY statement above is the display window paramter, which determines the final display range is [2017-11-03T00:00:00, 2017-11-03T23:00:00].

Then the system will use the time and value filtering condition in the WHERE clause and the second parameter of the GROUP BY statement as the data filtering condition to obtain the data satisfying the filtering condition (which in this case is the data in the range of (2017-11-03T06:00:00, 2017-11-03T23:00:00] and satisfying root.ln.wf01.wt01.temperature > 20), and map these data to the previously segmented time axis (in this case there are mapped data in every 1-day period from 2017-11-03T00:06:00 to 2017-11-03T23:00:00).

Since there is  no data in the result range [2017-11-03T00:00:00, 2017-11-03T00:06:00], the aggregation results of this segment will be null. There is data in all other time periods in the result range to be displayed. The execution result of the SQL statement is shown below:
![](./chapter4-fig/4.13.jpg)

It is worth noting that the path after SELECT in GROUP BY statement must be aggregate function, otherwise the system will give the corresponding error message, as shown below:
![](./chapter4-fig/4.14.jpg)

### 4.4.3 Index Query (Experimental Function)
Indexing time series can speed up some queries of this column (such as accelerating aggregate queries), or add new query functions to the column (such as similarity matching introduced in Section 4.4.3.1 of this chapter). User's operations mainly include the establishment, deletion and query of time series.

The present 0.7.0 version of IoTDB supports KvIndex, which is used to speed up the query of time series subsequence similarity matching.

#### 4.4.3.1 KvIndex (Similarity Matching Query)
In the field of timeseries data, similarity matching is a very common requirement. Users specify a time series subsequence as a pattern, hoping to find in another longer time series all subsequences similar to the specified pattern and return. As shown in Figure 4.3, for a given pattern, there are two subsequences (marked red) similar to the given pattern in a longer time series. Similarity matching requires that the two subsequences be found and returned. KvIndex is the index used to solve the problem of similarity matching.

![](./chapter4-fig/4.15.jpg)
<center>** Figure 4.3  KvIndex similarity matching query **</center>

##### 4.4.3.1.1 KvIndex Establishment
When created, KvIndex divides the time series into a series of equal-length blocks (the block length is parameterized as `window_length` in table 4-2) and creates indexes for the data after a timestamp (the timestamp is parameterized as `since_time` in table 4-2). Note that the time series must already exist before indexing. Successful execution will result in an "execute successfully" prompt that represents the completion of time series index establishment.

Note: After the index establishment, flush operation is required for the index to take effect.

<center>** Table 4-2 KvIndex establishment paramter list **</center>

|Parameter name (case insensitive)|Interpretation|
|:---|:---|
|window_length|INT32 type; mandatory; KvIndex divides the time series into a series of equal-length blocks, whose length is 'window_length'.|
|since_time|INT32 type; mandatory; create indexes for data larger than the timestamp 'since_time'; default value 0, i.e., index all data.|

See Section 7.1.4.1 for detailed SQL statements of establishing KvIndex. Here we give a practical example:
```
CREATE INDEX 
    ON root.ln.wf01.wt01.status 
    USING kvindex 
        WITH window_length=3, since_time=1000;
```
The above example creates a Kvindex for the time series with path root. ln. wf01. wt01. status, and the parameters required for the index are listed after the keyword WITH. Take KvIndex as an example, the following parameters are included: the length of the index block is 3, and the data with the timestamp greater than 1000 is indexed.

##### 4.4.3.1.2 KvIndex Query
After creating the KvIndex, the user can specify a pattern sequence and specify the KvIndex index query (i.e. similarity matching) for the data within a certain time range of a sequence. See Section 7.1.4.3 of this manual for detailed SQL statements.

Let's explain with a concrete example.
```
SELECT 
    KvIndex(root.ln.wf02.wt02.status,1,100,0.5,1.0,2.5) 
FROM
    root.ln.wf01.wt01.status 
WHERE 
    time >= 500 
    and 
    time <= 1000
;
```
The above example queries data in the column `root.ln.wf01.wt01.status` over a time range of [500, 1000]. Users use the fragment of the `root.ln.wf02.wt02.status` sequence within a time range of [1,100] as a pattern.

For queries on the KvIndex, the time range must be a continuous time period, i.e., the time condition can only be in the following three forms:

(1) time > a, e.g., time > 1000;

(2) time < b, e.g., time < 2000;

(3) time > a and time < b, e.g., time > 1000 and time < 2000.

KvIndex does not support other forms of time condition such as "time > 1000 and time < 2000 and time > 3000".

Users can also specify the remaining three parameters, namely:

(1) KVIndex requires that the Euclidean distance between the pattern and the matching subsequence be less than epsilon, i.e. the epsilon parameter in Table 4-3, which is a mandatory parameter. In the above example, epsilon=0.5;

(2) Since there may be a mean shift between the pattern and the matched subsequence, the user is allowed to specify the mean threshold, i.e., the alpha parameter in Table 4-3, which is an optional parameter with a default value of 0. In the above example, alpha=1.0.

(3) Because there may be a standard deviation shift between the pattern and the matched subsequence, the user is allowed to specify the standard deviation threshold, i.e., the beta parameter in Table 4-3, which is an optional parameter with a default value of 1. In the above example, beta=2.5;

Detailed descriptions of all parameters are given in Table 4-3.

<center>** Table 4-3 KvIndex query paramter list **</center>

|Parameter name (case insensitive)|Interpretation|
|:---|:---|
|pattern_path|the time series that the pattern belongs to|
|pattern_start_time|INT64; the start timestamp of the pattern|
|pattern_end_time|INT64; the end timestamp of the pattern|
|epsilon|DOUBLE; Euclidean distance threshold between the pattern and the matching subsequence|
|alpha|DOUBLE; mean threshold; optional; default value 0|
|beta|DOUBLE; standard deviation threshold; optional; default value 1|

The matching results are sorted in ascending order according to Euclidean distance and returned. The returned result is a triple, including start time, end time and Euclidean distance. The query result of the above example is shown below:
![](./chapter4-fig/4.16.jpg)

##### 4.4.3.1.3 KvIndex Deletion
For a time series that has created a KvIndex, the user can delete the time series. Users need to specify the name of the time series to be deleted. See Section 7.1.4.2 of this manual for detailed SQL syntax.

Here we give an example of deletion:
```
DROP INDEX kvindex ON root.ln.wf01.wt01.status;
```
The above example deletes the KvIndex of the time series with the path `root.ln.wf01.wt01.status`. Successful execution will result in an "execute successfully" prompt that represents the completion of time series index deletion.

###  4.4.4 Automated Fill
In the actual use of IoTDB, when doing the query operation of time series, situations where the value is null at some time points may appear, which will obstruct the further analysis by users. In order to better reflect the degree of data change, users expect missing values to be automatically filled. Therefore, the IoTDB system introduces the function of Automated Fill.

Automated fill function refers to filling empty values according to the user's specified method and effective time range when performing time series queries for single or multiple columns. If the queried point's value is not null, the fill function will not work.

Note: In the current version 0.7.0, IoTDB provides users with two methods: Previous and Linear. The previous method fills blanks with previous value. The linear method fills blanks through linear fitting. And the fill function can only be used when performing point-in-time queries.

#### 4.4.4.1 Fill Method
##### 4.4.4.1.1 Previous Method
When the value of the queried timestamp is null, the value of the previous timestamp is used to fill the blank. The formalized previous method is as follows (see Section 7.1.3.6 for detailed syntax):
```
select 
    <path> 
from 
    <prefixPath> 
where 
    time = <T> 
fill
    (<data_type>[previous, <before_range>], …)
```

Detailed descriptions of all parameters are given in Table 4-4.

<center>** Table 4-4 Previous fill paramter list **</center>

|Parameter name (case insensitive)|Interpretation|
|:---|:---|
|path, prefixPath|query path; mandatory field|
|T|query timestamp (only one can be specified); mandatory field|
|data_type|the type of data used by the fill method. Optional values are int32, int64, float, double, boolean, text; optional field|
|before_range|represents the valid time range of the previous method. The previous method works when there are values in the [T-before_range, T] range. When before_range is not specified, before_range takes the default value T; optional field|

Here we give an example of filling null values using the previous method. The SQL statement is as follows:
```
select 
    temperature 
from 
    root.sgcc.wf03.wt01 
where 
    time = 2017-11-01T16:37:50.000 
fill
    (float[previous, 1m]) 
```
which means:

Because the time series root.sgcc.wf03.wt01.temperature is null at 2017-11-01T16:37:50.000, the system uses the previous timestamp of 2017-11-01T16:37:50.000 (and the timestamp is in the [2017-11-01T16:36:50.000, 2017-11-01T16:37:50.000] time range) for fill and display.

On the sample data given in Section 4.1.2 of this chapter, the execution result of this statement is shown below:
![](./chapter4-fig/4.17.jpg)

It is worth noting that if there is no value in the specified valid time range, the system will not fill the null value, as shown below:
![](./chapter4-fig/4.18.jpg)

##### 4.4.4.1.2 Linear Method
When the value of the queried timestamp is null, the value of the previous and the next timestamp is used to fill the blank. The formalized linear method is as follows (see Section 7.1.3.6 for detailed syntax):
```
select 
    <path> 
from 
    <prefixPath> 
where 
    time = <T> 
fill
    (<data_type>[linear, <before_range>, <after_range>]…)
```

Detailed descriptions of all parameters are given in Table 4-5.

<center>** Table 4-5 Linear fill paramter list **</center>

|Parameter name (case insensitive)|Interpretation|
|:---|:---|
|path, prefixPath|query path; mandatory field|
|T|query timestamp (only one can be specified); mandatory field|
|data_type|the type of data used by the fill method. Optional values are int32, int64, float, double, boolean, text; optional field|
|before_range, after_range|represents the valid time range of the linear method. The previous method works when there are values in the [T-before_range, T+after_range] range. When before_range and after_range are not explicitly specified, both before_range and after_range default to infinity; optional field|

Here we give an example of filling null values using the linear method. The SQL statement is as follows:
```
select 
    temperature 
from 
    root.sgcc.wf03.wt01 
where 
    time = 2017-11-01T16:37:50.000 
fill
    (float [linear, 1m, 1m])
```
which means:

Because the time series root.sgcc.wf03.wt01.temperature is null at 2017-11-01T16:37:50.000, the system uses the previous timestamp 2017-11-01T16:37:00.000 (and the timestamp is in the [2017-11-01T16:36:50.000, 2017-11-01T16:37:50.000] time range) and its value 21.927326, the next timestamp 2017-11-01T16:39:00.000 (and the timestamp is in the [2017-11-01T16:36:50.000, 2017-11-01T16:37:50.000] time range) and its value 25.311783 to perform linear fitting calculation: 21.927326 + (25.311783-21.927326)/60s*50s = 24.747707

On the sample data given in Section 4.1.2, the execution result of this statement is shown below:
![](./chapter4-fig/4.17.jpg)

It is worth noting that if there is no value in the specified valid time range, the system will not fill the null value, as shown below:
![](./chapter4-fig/4.18.jpg)

#### 4.4.4.2 Correspondence between Data Type and Fill Method
Data types and the supported fill methods are shown in Table 4-6.

<center>** Table 4-6 Data types and the supported fill methods **</center>

|Data Type|Supported Fill Methods|
|:---|:---|
|boolean|previous|
|int32|previous, linear|
|int64|previous, linear|
|float|previous, linear|
|double|previous, linear|
|text|previous|


It is worth noting that IoTDB will give error prompts for fill methods that are not supported by data types, as shown below:
![](./chapter4-fig/4.19.jpg)

When the fill method is not specified, each data type bears its own default fill methods and parameters. The corresponding relationship is shown in Table 4-7.

<center>** Table 4-7 Default fill methods and parameters for various data types **</center>

|Data Type|Default Fill Methods and Parameters|
|:---|:---|
|boolean|previous, 0|
|int32|linear, 0, 0|
|int64|linear, 0, 0|
|float|linear, 0, 0|
|double|linear, 0, 0|
|text|previous, 0|

Note: In version 0.7.0, at least one fill method should be specified in the Fill statement.








