# Chapter 4 IoTDB Operation Manual

## 4.1 Scenario Description and Sample Data

To make this manual more practical, we will use a specific scenario example in this chapter to illustrate how to operate IoTDB databases at all stages of use. For your convenience, we also provide you with a sample data file in the scenario of this chapter for you to import into the IoTDB system for trial and operation.

### 4.1.1 Scenario Description
A power department needs to monitor the operation of various power plants under its jurisdiction. By collecting real-time monitoring data sent by various types of sensors deployed by various power plants, the power department can monitor the real-time operation of the power plants and understand the trend of data changes, etc. IoTDB has the characteristics of high write throughput and rich query functions, which can provide effective support for the needs of the power department.

The real-time data needed to be monitored involves multiple attribute levels:

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

##4.2 Data Model Selection and Creation
Before importing data to IoTDB, we first select the appropriate data storage model according to the sample data provided in Section 4.1, and then set up the storage group and create time series using SET STORAGE GROUP statement and CRATE TIMESERIES statement respectively (see Section 7.1.2.2 of this manual for detailed grammar).











