<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [**Overview of Differences**](#overview-of-differences)
- [**Key Distinctions**](#key-distinctions)
    - [**1. Scope and Focus**](#1-scope-and-focus)
    - [**2. Detail vs Performance**](#2-detail-vs-performance)
    - [**3. Usage**](#3-usage)
    - [**4. Data Destination**](#4-data-destination)
- [**Summary**](#summary)
  - [Trace Node](#trace-node)
- [Validation settings for input type nodes](#validation-settings-for-input-type-nodes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Monitoring and Troubleshooting

In IBM App Connect Enterprise (ACE), Activity Logs, Resource Statistics, Trace, and Business Transaction Monitoring (BTM) are distinct tools used for different aspects of monitoring and troubleshooting, primarily differing in their level of detail and purpose. The different monitoring features provide distinct levels and types of information, ranging from high-level overviews to detailed debugging data and business-focused tracking. 

---

# **Overview of Differences**

| **Feature**                               | **Purpose**                                                                                            | **Data Type**                                                             | **Detail Level**                                     | **Performance Impact**                                    |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------- |
| **Activity Logs**                         | High-level overview of interactions with external resources for initial investigation.                 | Concise, tagged log entries (text).                                       | High-level; avoids technical complexity.             | **Low** – enabled by default.                             |
| **Resource Statistics**                   | Quantitative data on the performance and usage of internal resources (CPU, memory, specific nodes).    | Metrics and performance data (quantitative).                              | System/resource-level metrics.                       | **Moderate** – requires activation.                       |
| **Trace (User and Service)**              | Detailed debugging and problem determination by following internal code paths.                         | Extensive, human-readable logs of internal execution and message content. | Extremely detailed; captures message data snapshots. | **High** – significant performance impact; use sparingly. |
| **Event Monitoring**                      | Emits custom events at significant points in a flow for external systems (auditing, monitoring tools). | Configurable events (XML or JSON).                                        | Configurable points within a flow.                   | **Moderate** – depends on config & throughput.            |
| **Business Transaction Monitoring (BTM)** | Tracks the lifecycle and status of a business transaction across multiple flows.                       | Correlated monitoring events presented as transaction summaries.          | Business-process level; transaction-oriented.        | **Moderate** – uses event monitoring + database.          |

---

# **Key Distinctions**

### **1. Scope and Focus**

* **Activity Logs** & **Trace** focus on *technical behaviour* of ACE.

  * Activity Logs → operational overview
  * Trace → deep technical debugging
* **Resource Statistics** → performance monitoring & bottleneck detection
* **Event Monitoring** & **BTM** → focused on *data and business processes*

  * Event Monitoring → significant flow events
  * BTM → end-to-end business transaction lifecycle

---

### **2. Detail vs Performance**

* **Trace** gives maximum detail but has the **highest performance impact** → use only when needed.
* **Activity Logs** are concise, lightweight, safe for continuous use.
* **Resource Statistics** give important platform metrics with moderate overhead.
* **Event Monitoring/BTM** add business visibility with manageable cost.

---

### **3. Usage**

* **Activity Logs** → first step when investigating unexpected behaviour.
* **Trace** → last resort for complex or unexplained issues.
* **Resource Statistics** → performance tuning, detecting resource issues.
* **Event Monitoring** → external audit, logging, dashboards.
* **BTM** → monitoring full business workflows and SLAs.

---

### **4. Data Destination**

* **Activity Logs** → UI, log files, ELK/Splunk.
* **Resource Statistics** → UI or stats files.
* **Trace** → local trace files only.
* **Event Monitoring** → MQ topics or BTM Recorder.
* **BTM** → database for correlated transaction views.

---

# **Summary**

You use:

* **Activity Logs** → quick operational overview
* **Resource Statistics** → performance & tuning
* **Trace** → deep technical debugging
* **Event Monitoring** → custom visibility and external processing
* **BTM** → tracking business outcomes end-to-end

---

If you'd like, I can convert this into a **colour-coded table**, **ACE interview answer**, or **visual diagram**.


## Trace Node

This is the default logging mechanism in IIB. In the Trace node there are 3 types of trace destination.

![Trace Node](../images/traceNode.png)

__File__

You enter the <path>/<file>. Then you give the pattern made up of user text and ESQL expressions in curly braces. ${Root} is the entire message tree.
```
Message Tree ${Root}
-----------------------
OriginalPayload :${Root.XMLNSC.k.l}
-----------------------
```

You could MQPUT XML on the IN queue:
```
<k><l>welcome</l></k>
```
The disadvantage is that you cannot switch off the file trace on the fly and the destination file will increase in size taking up PROD disk space. Because of this disk space issue the User Trace comes into the picture.

__User Trace__

No need for a file path since the trace events are going to the broker memory then whenever required we can download those logs.
```
mqsichangetrace IIBGURU -e IIBGURU_EX -u -l debug -f myMsgFlow // switches it on. The -u means user trace
mqsireadlog IIBGURU -e IIBGURU_EX -u -o usertracelog.txt // This downloads the logs
mqsiformatlog -i usertracelog.txt -o usertracelog.xml // change to XML format
mqsichangetrace IIBGURU -e IIBGURU_EX -u -l none
```
[DEC302020-IIB RoutingTryCatchTrace 28 minutes](https://drive.google.com/file/d/1OtA5Kyv5lCXJLzgyIIjcrndxdXGaKBOM/view?usp=share_link) Logs all validation
failures to the user trace, even if you have not asked for user tracing
of the message flow. Use this setting if you want processing of the
message to continue regardless of validation failures.

**Local Error Log**

Logs all validation failures to the error log (for example, the Event
Log on Windows). Use this setting if you want processing of the message
to continue regardless of validation failures.

# Validation settings for input type nodes

**Exception**

The default value. An exception is thrown on the first validation
failure encountered. The resulting exception list is shown below. The
failure is also logged in the user trace if you have asked for user
tracing of the message flow, and validation stops. Use this setting if
you want processing of the message to halt as soon as a failure is
encountered.

**Exception List**

Throws an exception if validation failures are encountered, but only
when the current parsing or writing operation has completed. The
resulting exception list is shown below. Each failure is also logged in
the user trace if you have asked for user tracing of the message flow,
and validation stops. Use this setting if you want processing of the
message to halt if a validation failure occurs, but you want to see the
full list of failures encountered. This property is affected by the
Parse Timing property; when partial parsing is selected the current
parsing operation parses only a portion of an input message, so only the
validation failures in that portion of the message are reported.

[← Back to Main page](../IIB_ACE.md)