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

**Table 4-1 The basic information of the data**

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

![](./chapter4-fig/4.1.jpg)

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
select status, temperature from root.ln.wf01.wt01 
where 
time > 2017-11-01T00:05:00.000 and time < 2017-11-01T00:12:00.000
```
which means:

The selected device is ln group wf01 plant wt01 device; the selected time series is "status" and "temperature". The SQL statement requires that the status and temperature sensor values between the time point of "2017-11-01T00:05:00.000" and "2017-11-01T00:12:00.000" be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.7.jpg)

#### 4.4.1.3 Select Multiple Columns of Data for the Same Device According to Multiple Time Intervals
IoTDB supports specifying multiple time interval conditions in a query. Users can combine time interval conditions at will according to their needs. For example, the SQL statement is:
```
select status,temperature from root.ln.wf01.wt01 
where 
    (time > 2017-11-01T00:05:00.000 and time < 2017-11-01T00:12:00.000) 
or 
    (time >= 2017-11-01T16:35:00.000 and time <= 2017-11-01T16:37:00.000)
```
which means:

The selected device is ln group wf01 plant wt01 device; the selected time series is "status" and "temperature"; the statement specifies two different time intervals, namely "2017-11-01T00:05:00.000 to 2017-11-01T00:12:00.000" and "2017-11-01T16:35:00.000 to 2017-11-01T16:37:00.000". The SQL statement requires that the values of selected time series satisfying any time interval be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.8.jpg)

#### 4.4.1.4 Choose Multiple Columns of Data for Different Devices According to Multiple Time Intervals
The system supports the selection of data in any column in a query, i.e., the selected columns can come from different devices. For example, the SQL statement is:
```
select wf01.wt01.status,wf02.wt02.hardware from root.ln 
where 
    (time > 2017-11-01T00:05:00.000 and time < 2017-11-01T00:12:00.000) 
or 
    (time >= 2017-11-01T16:35:00.000 and time <= 2017-11-01T16:37:00.000)
```
which means:

The selected timeseries are "the power supply status of ln group wf01 plant wt01 device" and "the hardware version of ln group wf02 plant wt02 device"; the statement specifies two different time intervals, namely "2017-11-01T00:05:00.000 to 2017-11-01T00:12:00.000" and "2017-11-01T16:35:00.000 to 2017-11-01T16:37:00.000". The SQL statement requires that the values of selected time series satisfying any time interval be selected.

The execution result of this SQL statement is as follows:
![](./chapter4-fig/4.9.jpg)

### 4.4.2 Down-Frequency Aggregate Query

