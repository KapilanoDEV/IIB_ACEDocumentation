







# IIB Node Connecting to Database using ODBC pt1

Use the command console to associate a DSN with a broker instance. You
create an ODBC connection to the data source DBNAME with the
command:

```
mqsisetdbparms IB9BROKER -n DBNAME -u -p
```

You now need to reload the configuration since the new association of DBNAME to IB9BROKER:

```
mqsireload IB9BROKER
```

Then check the connection using 
```
mqsicvp IB9BROKER -n DBNAME
```

# IIB Node Connecting to Database using ODBC pt2

IIBGURU entered:

```
$ db2start


$ db2
db2 => connect to sample
db2 => SELECT * FROM EMPLOYEE
```


Create a new Application name DBINTER_APP. Then right click on
DBINTER_APP then create a message flow DBINTER_MF. From the palette add
MQInput and MQOutput nodes. In between add some ESQL in a Compute node.
MQInput out connects to the in of the Compute node. Compute out connects
to the in of the MQOutput node. MQInput node will use the XMLNSC parser
to read XML records

```
        9001
        TRUMP
        New York
```

...that will be INSERTED into DBNAME. The Compute node has Data Source
DBNAME. Double click on the node to bring up the ESQL
editor.

Uncomment --- CALL CopyMessageHeaders(); so that we can send a success
message to the out queue.
Compare this SQL

```
INSERT INTO EMPLOYEE VALUES(101,'IIBGURU','CALIFORNIA')
```

With what we use in ESQL Compute Node.

```
INSERT INTO Database.iibguru.EMPLOYEE
VALUES(InputRoot.XMLNSC.EMPLOYEE.ENO, InputRoot.XMLNSC.EMPLOYEE.ENAME, InputRoot.XMLNSC.EMPLOYEE.ECITY);
```

> **Note** You can use CTRL space to have content assist help with the syntax

The above could be written as:

```
  DECLARE InRef REFERENCE TO InputRoot.XMLNSC.EMPLOYEE;
  INSERT INTO Database.iibguru.EMPLOYEE VALUES(InRef.ENO, InRef.ENAME, InRef.ECITY);
```

We are getting the values from the Input message tree where the domain
is XMLNSC, EMPLOYEE is the root child. "Database" is a keyword & the
iibguru is the schema in which EMPLOYEE table exists. Here the schema is
the user name. It will use the user name as the schema if you have not
created the schema before creating a table in the database.

When you save the ESQL in the Compute node you get a warning
"Unresolvable database table reference 'Database.iibguru.EMPLOYEE'".
This is because it only resolves and finds EMPLOYEE during execution.
The successive command after the INSERT will only run if the INSERT
executes correctly. It is:

```
  SET OutputRoot.XMLNSC.DBINSERT.STATUS='SUCCESS';
```



Introduce a breakpoint after the MQInput. Since you have the breakpoint
you have to launch the debugger (right click on execution group) then
add a message from the XML file using RFHUTIL. In the Data tab of the
RFHUTIL you can parse the file to see if it is syntactically correct
under Data Format. Once you write the XML file to the IN queue the Debug
perspective is automatically launched. You will see multiple threads.
Thread suspended at DBINTER_MF at connection. You can click on the 'Edit
Source Lookup Path' button. From there you add a container to the source
lookup path. Since we are running a MF we choose the Message Flow
Container, choose our DBINTER_APP application which contains our message
flow. Under Variables you can see the Message. As you know the Message
has Properties then MQMD then Other headers then the Body which
has



```
  XMLNSC
  |
  |
  V
  EMPLOYEE
    |
    |
    V
    ENO (9001), ENAME (TRUMP), ECITY(New York)
```

We can go inside the scope of the Compute node by clicking on the
thread in the Debug perspective, then click on Step into code icon which
will open up the ESQL of the next Compute node. You will reach the CALL
CopyMessageHeaders(); You can then click on the Step over icon which
will run the CALL then take you to the INSERT. The CALL will copy the
headers to the output. EVEN though the INSERT has been executed the
change has not been committed to the database. This is because
the

```
INSERT INTO....
SET OutputRoot...
RETURN TRUE;
```

Are considered as 1 unit of work. Only until every command in the ESQL
happens without error & the ___whole flow___ completes will the
INSERT be committed. Else a rollback will occur. This is controlled by
the Transaction property of the Compute node. The possible values are
Automatic (default) or Commit. The latter will commit the INSERT once
the Compute node completes successfully. It will not wait for the whole
flow to complete.



***PATH AREA.CIRCLE*** this allows you to reference ESQL in another broker schema



# Oracle DB ESQL

DSN TEST_DSN, Schema is SYSTEM

```
DECLARE EMP ROW;

--- ROW is a datatype that holds a tree structure]

SET EMP.Result[] = SELECT * FROM Database.SYSTEM.EMP_DETAILS AS R WHERE R.EMPID=InRef.id;
```


We then send this XML:
```
SET OutputRoot.JSON.Data.Employee.ID = EMP.Result.EMPID; //only works with a single value not multiple
```

# PASSTHRU with EVAL

[EVAL function in ESQL](https://youtu.be/0CX9--Y2jOk?si=0Mx5cJC8IDdW2GL1)

Example of PASSTHRU where there are 2 procedures with the same name but
in ESQL files contained in different schemas. How can we invoke them
dynamically at runtime? We can use EVAL to run the procedure that is
connected to a key value in some incoming XML or JSON. For example if
the value is Professional then invoke the EmpDetails procedure in the
Pro schema. If it is Personal invoke the EmpDetails in the Personal
schema:

```
SET Emp.EmpRecord[] = PASSTHRU('SELECT * FROM EMPLOYEE WHERE EMPNO='||''''||EmpID||'''');
SET Emp.EmpRecord[] = PASSTHRU('SELECT * FROM EMPLOYEE WHEREEMPNO='||''''||EmpID||'''');

DECLARE DbRecords REFERENCE TO Emp.EmpRecord;

CREATE FIELD OutputRoot.JSON.Data.Employee.{EmpInFoType};

DECLARE outDetailsRef REFERENCE TO OutputRoot.JSON.Data.Employee.{EmpInFoType};

IF CONTAINS(EmpInFoType, 'Professional') THEN SET SchemaName = 'EmpProDet';
ELSEIF CONTAINS(EmpInFoType, 'Personal') THEN SET SchemaName = 'EmpPersonDet';
END IF;

EVAL('CALL '||SchemaName||'.EmpDetails(outDetailsRef, DbRecords);');

RETURN TRUE;
```

This video also shows you how to use EVAL.
[EVAL function in ESQL || Explained with two Practical Examples](https://youtu.be/0CX9--Y2jOk?si=h0r2PTL2h-y_qw-x). EVAL saves you from having to write lots of if statements by evaluating the expression on the fly using variables.

Example:
```
SET OutputRoot.JSON.Data.Result = EVAL('num1'||mathop||'num2');
```

Instead of:

```
IF mathop = '+' THEN
  SET OutputRoot.JSON.Data.Result = num1 + num2;
```

# Connect DB JAVA type 4 driver

[JAN072021-IBM IIB REST_GET_SSL_DB 32 minutes](https://drive.google.com/file/d/1o6TpH129InsCw8LWumlPOqUrfjgZU7O_/view?usp=share_link). Before we connected using
the ODBC driver which is an operating system dependent driver. This is a
pure java driver that connects using the Java API calls. The DB vendors
like IBM and Oracle will provide the JAR files to connect to their
databases. In IIB you need to enable some settings in the Configurable
Services [JAN192021-IIB JCN_ForLoop&JDBC&ModellingPt1 8 minutes](https://drive.google.com/file/d/1dm5ErDsUbQ80Ln-EGWGrCF6UABv85uNE/view?usp=share_link). You can
make changes that will be registered to the broker (right click broker
in the toolkit, then select 'Start Web User interface'). Use the
"mqsireportproperties brokerName -c AllTypes -c
AllReportableEntityNames -r". From the output you can see the different
connectionUrlFormat for DB2, Oracle, MySQL etc.\
connectionUrlFormat='jdbc:db2://serverName:[portNumber]/databaseName:user=[user];password=[password];'
connectionUrlFormat='jdbc:oracle:thin:[user]/[password]@serverName:portNumber:[connectionUrlFormatAttr1]'\
JDBCProviders is the Configurable service that we need to Configure. We
also need type4DriverClassName='com.ibm.db2.jcc.DB2Driver' which we
will reference in the code. This class is available in the DB2 JAR
file.

You should also tick the exception option in the properties. For
example if you are using HTTP then the response will  contain the
error.

BIP2322E: Database error: SQL State ''IM002'';
Native Error Code '0'; Error Text
''[unixODBC][Driver Manager]Data source name not found and
no default driver specified

[ ]



# Deciding Between ODBC and JDBC Drivers

[ The following points may help: ]

Multithread: - JDBC is multi-threaded - ODBC is not multi-threaded (at
least not thread safe)

Flexibility: - ODBC is a windows-specific technology - JDBC is specific
to Java, and is therefore supported on whatever OS supports Java

Power : you can do everything with JDBC that you can do with ODBC, on
any platform.

Language: - ODBC is procedural and language independent - JDBC is object
oriented and language dependent (specific to java).

Heavy load: - JDBC is faster - ODBC is slower

ODBC limitation: it is a relational API and can only work with data
types that can be expressed in rectangular or two-dimensional format.
(it will not work with data types like Oracle's spatial data type)

API: JDBC API is a natural Java Interface and is built on ODBC, and
therefore JDBC retains some of the basic feature of ODBC
