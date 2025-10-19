<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Local, UDP (AKA External) & shared variables](#local-udp-aka-external--shared-variables)
- [Shared vs Static Library.](#shared-vs-static-library)
- [ESQL native to IIB](#esql-native-to-iib)
- [MOVE NEXTSIBLING vs CARDINALITY and [i] call in a WHILE loop](#move-nextsibling-vs-cardinality-and-i-call-in-a-while-loop)
- [The MOVE statement](#the-move-statement)
- [WHILE versus FOR loop](#while-versus-for-loop)
- [Converting XMLNSC to JSON](#converting-xmlnsc-to-json)
- [Difference between DECLARE varName CHAR FIELDNAME() & DECLARE varName REFERENCE TO](#difference-between-declare-varname-char-fieldname--declare-varname-reference-to)
- [LEAVE statement](#leave-statement)
- [COALESCE](#coalesce)
- [ESQL field reference overview](#esql-field-reference-overview)
- [Shared variable and ATOMIC block](#shared-variable-and-atomic-block)
- [Global Cache](#global-cache)
- [The THE function returns the first element of a list.](#the-the-function-returns-the-first-element-of-a-list)
- [DATE TIME TRANSFORMATION](#date-time-transformation)
- [INTERVAL DATATYPE](#interval-datatype)
- [Opaque Parsing](#opaque-parsing)
- [ASBITSTREAM in ESQL](#asbitstream-in-esql)
- [Aggregation](#aggregation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Local, UDP (AKA External) & shared variables

Local variable scope is within the Compute node. UDP scope is
throughout the message flow. Shared scope is through the execution group
AKA the Integration Server even if the message flow completes. When the
next transaction starts the shared variable is available.

# Shared vs Static Library.

30DEC2020-IIB RoutingTryCatchTrace 17 minutes. What is the difference
between a Static and Shared library in IBM Integration Bus? Libraries
can't include message flows only subflows. The subflows could take care
of say error handling. This is common to many applications. If you
deploy an application that references a shared library you see this
error BIP1301E: the application cannot be deployed since it references a
shared library that has not been deployed to the Integration Server
(drag and drop to the Integration Server). But a static library will not
complain since it is included in the deployed application.

[·]{.s11}[       ]{.s12}A. Shared libraries may include additional jar
files while Static libraries cannot.

[·]{.s11}[       ]{.s12}B. Shared libraries can be deployed to an
Integration Server while Static libraries cannot.

[·]{.s11}[       ]{.s12}C. Shared libraries encapsulate common code
that may be used by multiple applications while Static libraries do
not.

[·]{.s11}[       ]{.s12}D. Multiple applications can reference the same
Shared library without having to include the library as part of the
build as in the case of Static library.

[·]{.s11}[       ]{.s12}E. Static library appears under Included
Libraries folder. Shared libraries appear under Referenced folder.

[ ]

# ESQL native to IIB

Before you create your ESQL message flow you package it into a schema.
An example schema is com.test.tran. This concept is akin to JAVA
packages. An Integration Project's resources are reusable, but an
Application's resources are contained and hence isolated.

```
CREATE COMPUTE MODULE XMLToJsonMsgFlow_Compute
```

"CREATE MODULE" creates a module, which is a named container
associated with a node. Modules, functions & procedures contained by a
schema must all have unique names.
Modules for the Compute node, database node & filter node must all contain exactly 1 function called Main. This function should return a Boolean. Main is the entry point used by a message flow node when processing a message.

```
CREATE COMPUTE MODULE FileNode1_Compute
       CREATE FUNCTION Main() RETURNS BOOLEAN
       BEGIN

            -- CALL CopyMessageHeaders(); // This will only copy the Properties & MQMD. NOT the XMLNSC as I < J.
            -- CALL CopyEntireMessage();
            RETURN TRUE;

       END;

      CREATE PROCEDURE CopyMessageHeaders() 
      BEGIN
            DECLARE I INTEGER 1;
            DECLARE J INTEGER CARDINALITY(InputRoot.*[]);
            WHILE I < J DO
              SET OutputRoot.*[I] = InputRoot.*[I];
              SET I = I + 1;

            END WHILE;
      END;
END MODULE;
```

CARDINALITY returns the number of elements in a list; here the
correlation InputRoot is pointing to the message tree which contains the
Properties, MQMD and XMLNSC so CARDINALITY is 3. InputRoot is a field
reference with a [] array indicator.

InputRoot.\*[] allows you to refer to the array of all children of
the root element using a path element of \*.

InputRoot.\*[<] last child of root of the input message. As you can see from the [diagram](#converting-xmlnsc-to-json) it is Body. The last child element beneath the root
of the message tree is always the message body. For example, XMLNSC or
JSON.

InputRoot.\*[1] first child of the root of the input message i.e. the
message properties.

Remember that you cannot have more than 1 child element under XMLNSC.
If you have some code like this:

```
      SET OutputRoot.XMLNSC.Test...
      SET OutputRoot.XMLNSC.FINAL...
```

Then you get this in the ExceptionList "Unexpected XML type at this
point in document."

The **Environment Tree** is a special long lasting Tree that you can
both read from and write to.

# MOVE NEXTSIBLING vs CARDINALITY and [i] call in a WHILE loop

..with subscript code, where in order to find the [n+1]'th entry when
using indices, Broker must navigate to [1], [2], [3],....[n]
and then to [n+1]. Using subscripts is regarded as a performance no
no. Imagine you had 10,000 Order. This would result in each and every
iteration of the loop walking down the tree over the InputRoot, MRM,
ns:Data Nodes and then across the .ns:Order until it found the id and
ns:Row.sRow sibling nodes. This means by the time we get round to
processing id[10000] it will have had to repetetively walked over all
the other Order Nodes time and time again. In fact we can calculate it
would have walked over (2 x 10,000) + ((10,000 x (10,000 + 1)) / 2) =
50,035,000 Nodes.

```
DECLARE i INTEGER 1;
DECLARE orderCardinality INTEGER CARDINALITY(InputRoot.MRM.ns:Data.ns:Order[]);
WHILE i <= orderCardinality DO
   SET OutputRoot.MRM.Order[i].NMCIL=InputRoot.MRM.ns:Data.ns:Order[i].id;
   SET OutputRoot.MRM.Order[i].NUBSL=InputRoot.MRM.ns:Data.ns:Order[i].ns:Row.sRow;
   SET i = i + 1;
END WHILE;
```

Using NEXTSIBLING instead of indices to access the message tree will perform significantly better as the message tree grows larger. This code is faster to use when looping through messages. This is because using references to navigate trees is better performing. When using a reference, it just accesses the relevant pointer stored in the node datastructure. MOVE and LASTMOVE are used in combination. When MOVE is executed, the LASTMOVE checks if MOVE executed properly. So it's REFERENCE's to the rescue:

```
DECLARE i INTEGER 1;
DECLARE refOrder REFERENCE TO InputRoot.MRM.ns:Data.ns:Order[1];
WHILE LASTMOVE(refOrder) DO
   SET OutputRoot.MRM.Order[i].NMCIL=refOrder.CIL.id;
   SET OutputRoot.MRM.Order[i].NUBSL=refOrder.CIL.ns:Row.slRow;
   MOVE refOrder NEXTSIBLING NAMESPACE * NAME 'Order';               
   SET i=i+1;            
END WHILE;
```

> **Note** [ESQL Code Tips to Increase
Performance](https://ibmintegrationbus.wordpress.com/2019/06/03/esql-code-tips-to-increase-performance/) and [subscripts is regarded as a performance no
no](https://stackoverflow.com/questions/50554128/how-to-iterate-while-a-mapping-variables-from-environment-to-message-assembly-in)



# The MOVE statement

```
MOVE InMgrref NEXTSIBLING REPEAT TYPE NAME;
```

This moves InMgrref in the direction of the next sibling relative to
the current position if there is a sibling. The TYPE and NAME clauses
mean target is moved to a field with the given type "name". Fields that
do not match the criteria are skipped over. NAMESPACE * indicates any
namespace. MOVE and LASTMOVE are used in combination. When MOVE is
executed, the LASTMOVE checks if MOVE executed properly. If not LASTMOVE
returns false.

# WHILE versus FOR loop

You would use WHILE when the exit is not known to you. A FOR loop
extends from a FOR statement to an END FOR statement and executes for a
specified number of iterations, which are defined in the FOR statement.
For each iteration of the FOR loop, the iteration variable (declared as
i for example) is incremented or decremented. The main difference
between a WHILE loop and a FOR loop is that a FOR loop is guaranteed to
finish, but a WHILE loop is not. The FOR statement specifies the exact
number of times the loop executes, unless a statement causes the routine
to exit the loop. With WHILE, it is possible to create an endless loop.
The advantage of the FOR statement is that it iterates through a list
without your having to write any sort of loop construct (and eliminates
the possibility of infinite loops).

# Converting XMLNSC to JSON

If your ESQL has
```
SET OutputRoot.JSON.Data = InputRoot.XMLNSC;
```

This will only work if you are not sending multiple arrays of elements
in your XML.

![multiple arrays of elements](IIB.fld/ac126101.gif)

The same applies to JSON to XML. Your output root will not
have well formatted message trees. That means the nodes that are
downstream will fail while formatting the data
[JAN042021-IIBFileNodeBARMQReplyTryCatchFlowOrder](https://drive.google.com/file/d/15EypxxgHztb_VdEnJa82vgmFoQO7wY7L/view?usp=share_link)

# Difference between DECLARE varName CHAR FIELDNAME() & DECLARE varName REFERENCE TO

If your JSON looks like

```
{
  "root":
  {

  "Number1":4,

  "Number2":2,

  "MathOp":"+",

  }

}
```

As well as this code:

```
DECLARE InRef REFERENCE TO InputRoot.JSON.Data.root;
```

The InRef is a reference variable pointing to part of the message tree
(root).

```
DECLARE mathOp CHAR InRef.MathOp;
```

mathOp is +

```
DECLARE Operation CHAR FIELDNAME(InRef.*[<]);
```

The FIELDNAME returns the name of the field identified by
source_field_reference inside () as a character value. If the parameter
identifies a nonexistent field, NULL is returned. Operation is now
MathOp (a string of type CHAR).



# LEAVE statement

The LEAVE statement stops the current iteration of the containing
WHILE, REPEAT, LOOP, or BEGIN statement identified by Label. The
containing statement's evaluation of its loop condition (if any) is
bypassed and looping stops.

*Syntax*

```
LEAVE *Label*
```

*Examples*

In the following example, the loop iterates four times:

```
DECLARE i INTEGER;
SET i = 1;
X : REPEAT 
  ...
  IF i>= 4 THEN
    LEAVE X;
  END IF;

  SET i = i + 1;
UNTIL
  FALSE
END REPEAT;
```
# COALESCE

The __COALESCE__ function evaluates its parameters in
order and returns the first one that is not NULL.

# ESQL field reference overview

For example:
```
InputRoot.XMLNS.Data.Invoice
```

A field reference consists of a correlation name, followed by 0 or more
Path Elements separated by periods (.). The correlation name identifies
a well-known starting point and must be the name of a constant, a
declared variable, or one of the predefined start points; for example,
__InputRoot__. The InputRoot is the root of the input message. The
OutputRoot is the root of the output message. Message trees could refer
to the input message or output message. The tree structure created by
the parsers is independent of any message format (e.g. XML). The
exception to this is the subtree created as part of the message tree to
represent the message body. For example, XMLNSC, JSON or DFDL.

Message Tree structure is populated with the contents of the input
message bit stream. It has a root element InputRoot. Each tree is made
up of elements. Root has a number of child elements.

The Compute node has an input message assembly and at least 1 output
message assembly. Configure the Compute node to determine which trees
are included in the output message assembly. If you want the output
message assembly to contain a complete copy of the input message tree,
you can code a single ESQL SET statement to make the copy.

```
CREATE PROCEDURE CopyEntireMessage() BEGIN
  SET OutputRoot = InputRoot;
END;
```

This procedure copies the entire contents of the input message tree
(Properties and MQMD which are the header & XMLNSC which is the payload)
to the output message.

If you want the output message to contain a subset of the input message
tree, code ESQL to copy those parts only.

```
SET OutputRoot.XMLNS.ResAdd.sp1:addC=
InputRoot.XMLNSC.sp1:ReqAdd.sp1:intA + InputRoot.XMLNSC.sp1:ReqAdd.sp1:intB;
```

The message tree is always present and is passed from node to node in a
single instance of a message flow. The message tree includes all the
headers, in addition to the message body.

The header of the message tree is the Properties & MQMD. The payload is
the XMLNSC in the [diagram](#converting-xmlnsc-to-json) and is also called the Body tree which is a
structure of child elements that represents the message content (data)
and reflects the logical structure of that content. The body tree is
created by a body parser. Each element in the parsed tree is either a
name, value or name-value element. The FIELDTYPE field function returns
the type of a given field. Named field types must be used with the
capitalization shown.

The following types are domain-independent:

- Name
- Value
- NameValue
- MQRFH2.BitStream
- MQRFH2.Field
- MQRFH2C.Field

Inside the main function after the 2nd call:

```
CREATE LASTCHILD OF OutputRoot DOMAIN('JSON');
CREATE LASTCHILD OF headerRef TYPE NameValue NAME 'InvoiceNumber' VALUE 'InvoiceNumber';
CREATE FIELD OutputRoot.JSON.Data;
```

LASTCHILD is a type of FIELD clause. "CREATE LASTCHILD OF" target
navigates to the target field and adds a new field as it's rightmost
child, displacing the previous last child to the left. The
DOMAIN('JSON') associates the new field with a new parser of the
specified type i.e. JSON. The second example creates a field using the
specified type, name, and value.

```
DECLARE outref REFERENCE TO OutputRoot.JSON.Data;
 CREATE FIELD outref.Purchases IDENTITY (JSON.Array)Purchases;
SET outref.Purchases.Item[1].Description = inRef.Description;

-- This creates JSON message under the Data element owned by
JSON parser root. This creates a child element of the OutputRoot
-- as a JSON. Since the JSON tree structure is JSON and data
this assigns InputRoot.XMLNSC to OutputRoot.JSON.Data.
```

You can manipulate messages that belong to the JSON domain, which are
parsed by the JSON parser. The code transforms the OutputRoot message to
JSON. The OutputRoot... statement produces a message tree.

If you include a FIELD clause the field specified by TARGET is
navigated to.

SET assigns a value to a variable.

XMLNSC parser is guided by the XML Schema (describes the shape of the
message tree which is a logical model). XMLNSC is the preferred domain
for parsing all XML because of it's high performance, reduced memory
used by the logical message tree created from the parsed message; which
has discarded non-significant whitespace, mixed content, comments,
processing instructions & embedded DTDs. XMLNSC parser can operate as a
model-driven parser and can validate XML messages against XML schemas,
to ensure XML messages are correct.

# Shared variable and ATOMIC block

[What is Atomic block in ESQL?](https://youtu.be/u5-r4H6PvIY?si=kVKM9JrdJnXVlj6p)

```
DECLARE MYROW ROW; --A row variable contains an array
```

ROW SHARED variables can hold complex tree structure. A shared ROW can contain the result of a SELECT on the database table being cached.

```
DECLARE CACHE SHARED ROW; --For example, a database table called
"AIRPORTS" contains two columns, "CODE" and "CITY". This code loads the cache:
SET CACHE.AIRPORT[] = SELECT A.CODE, A.CITY FROM Database.AIRPORTS AS A;
```
The CACHE variable will be populated like this:

-- CACHE.AIRPORT[1].CODE = AAA

-- CACHE.AIRPORT[1].CITY = Anaa

-- CACHE.AIRPORT[2].CODE = AAB

-- CACHE.AIRPORT[2].CITY = Arrabury

Accessing ROW SHARED variables is much faster than retrieving the data
from the Database directly depending the amount of data retrieved. The
first time after the ESQL code is run CACHE shared variable is in
memory. The next time it is invoked it does not have to reload from the
database as it is cached.

The problem with this cache structure is that it doesn't scale. A user
trace will show that SELECT scans the table sequentially until it finds
a row that satisfies the WHERE clause. As the table grows, the search
gets slower. There comes a point when it's faster to drop the cache and
go to the database each time. 

> Ref:
[Efficient Caching](https://www.websphereusergroup.co.uk/wug/presentations/38/EfficientCaching\_-V2.pptx.pdf)

![Screenshot from Youtube vid](IIB.fld/SharedVar_Main.png)

Whatever is stored in a shared variable is held in cache memory until
we refresh the execution group, application or application flow using
'mqsireload' for example. Other message flows in the same execution
group access the shared row variable value set in a message flow.

Once a cached shared row is populated then every time the flow will run
it will only have what was loaded at the time of the SQL query. This
means that the underlying database values may change but this won't be
reflected.

In a PROD environment you can't run 'mqsireload' all the time as that
would impact LIVE services. One option is to create a CACHE queue on the
queue manager. This could be referenced in the properties of the same
flow but in a separate MQ Input node connected to a Compute node. Within
the Compute node you can empty the shared row variable:

```
SET CACHE = NULL;
```

When ESQL code in other compute nodes access the variable, they will
see it is empty, which prompts another SELECT against the database. If
the "SET CacheTable = NULL;" is executed just before "SET
OutputRoot.XMLNSC.Data = CacheTable;" then the output message assembly
will have no data.

To get around this use atomic blocks [(reference the link at the beginning of this header)](#shared-variable-and-atomic-block)
 within the same
schema to ensure that threads are executed serially (only 1 thread is
executed at a time). That way we can update the shared row variable in a
thread safe manner.


shared_atomic_Refresh_Cache.esql
```  
  BEGIN
    X: BEGIN ATOMIC
      SET CacheTable = NULL;  --Empties CacheTable
    END X;
      RETURN TRUE;
  END;
```

shared_atomic_Compute.esql

```
  BEGIN
    X: BEGIN ATOMIC
      IF NOT EXISTS(CacheTable.[]) THEN -- Does not check if CACHE exists but if data is in it.
        SET CacheTable.Result[] = PASSTHRU('SELECT R.FIRSTNME,R.LASTNAME FROM EMPLOYEE AS R');
        SET OutputRoot.XMLNSC.Data = CacheTable;
      
      ELSE
        SET OutputRoot.XMLNSC.Data = CacheTable;
      END IF;
    END X;
    RETURN TRUE;
  END;
```

Both atomic blocks must use the same X label. Now the CacheTable cache will not be refreshed just before setting the output root since the X atomic blocks have to run one after the other in their entirety.

One advantage of the Global Cache over ESQL shared variables is that
the cache can be shared between message flows, integration servers
execution groups, and integration brokers (recall that the scope
of ESQL shared variables is the message flow). However, because Global
Cache uses the JVM to store data (Udemy 18) it is slower than shared
variables which use cache.

# Global Cache

[One of the most important interview questions. Udemy 17]

[ ]

Shared variables used to store a temporary variable in Cache available
to all message flows in the same broker schema.

Supposing you want variables to be accessible across schemas. Clearly
Shared variables will not help you.

Global Cache allows you to share variables across schemas AND execution
groups or Integration node (broker).

[ ]

[mqsireportproperties IIBGURU -b cachemanager -o CacheManager -r]

[ ]

[CacheManager]

[  uuid='CacheManager']

[  policy='disabled']

[  portRange='2840-2859']

[  listenerHost='']

[  shutdownMode='fast']

[  objectGridCustomFile='']

[  deploymentPolicyCustomFile='']

[ ]

[BIP8071I: Successful command completion.]

mqsichangeproperties IIBGURU -b cachemanager -o CacheManager -n
policy,portRange,listenerHost -v default,generate,localhost

[ ]

This command will only allow variables to be shared within the
Integration node (broker) only.

[ ]

Two concepts you need to be aware of. Catalog server and container
server.

[ ]

Suppose I deploy an application in the default Integration server (EG)
that will LOAD a variable in Global Cache.

Note that default is a primary server that contains all the Global
Cache variables AKA a Catalog server.

Imagine that on EG2 I deploy an application that will RETRIEVE a
variable in Global Cache.

EG2 is a container server meaning it will load a local copy of the
Global Cache held on the primary EG namely default. If you have 2
Integration Servers acting as catalog servers then you have some form of
high availability

[ ]

If I run 'mqsireload default' then the values present in my Global
Cache will be gone because the Global Cache will be reloaded.

[This will also happen if I restart the broker.]

[ ]

[Enabling Global Cache for multiple Integration Nodes (brokers)]

[----------------------------------------]

You need a policy.xml file that will contain the names of the nodes.
There are some samples under
~/iib-10.0.0.22/server/sample/globalcache.

[ ]

[You have to run the command for all brokers.]

[ ]

[mqsichangebroker BROKER1 -b \<PATH>/policy_two_brokers.xml]

[mqsichangebroker BROKER2 -b \<PATH>/policy_two_brokers.xml]

[ ]

[mqsichangebroker BROKER2 -b disabled //This will switch it off]

[ ]

[----------------------------------------]

A static method means that it can be accessed without creating an
object of the class, unlike public

We might have different EG accessing an XML in Global Cache. We will
need an ESQL code to invoke some JAVA code.

mqbrkruser@SATELLITE-L50D-B:~$ cat
IBM/IIBT10/workspace/GlobalCacheApp/LoadGlobalCache/SubFlow/LoadGlobalCache_LoadCache.esql

[BROKER SCHEMA LoadGlobalCache.SubFlow]

[CREATE COMPUTE MODULE LoadGlobalCache_LoadCache]

[]CREATE FUNCTION Main() RETURNS BOOLEAN

[]BEGIN

[DECLARE InBlob BLOB;]

SET InBlob = ASBITSTREAM(InputRoot.XMLNSC,
InputRoot.Properties.Encoding, InputRoot.Properties.CodedCharSetId); --
Converts XML to BLOB so that it can be added to the Global Cache

[CALL LoadXMLToCache('GBKey', InBlob);]

[SET OutputRoot.XMLNSC.Message.Value='Successfully Loaded';]

[RETURN TRUE;]

[]END;

[]CREATE PROCEDURE LoadXMLToCache (IN key CHARACTER, IN Value BLOB
)

[]LANGUAGE JAVA

[]EXTERNAL NAME "GlobalCacheClass.LoadXMLToCache";

[END MODULE;]

mqbrkruser@SATELLITE-L50D-B:~$ cat
IBM/IIBT10/workspace/GlobalCache_PRJ/src/GlobalCacheClass.java

[import com.ibm.broker.plugin.MbException;]

[import com.ibm.broker.plugin.MbGlobalMap;]

[public class GlobalCacheClass {]

[public static String MapName = "GBMAP";]

public static void LoadXMLToCache(String key, byte[] xml) {

[try {]

// getGlobalMap is a static method. Static methods can be called
without creating objects MbGlobalMap map =
MbGlobalMap.getGlobalMap(MapName);

[]if(map.containsKey(key)){

[ map.update(key, xml);]

[}]

[else {]

[map.put(key, xml);]

[}]

[} catch (MbException e) {]

[e.printStackTrace();]

[}]

[}]

public static byte[] GetCacheValue(String key) {

byte[] xml = null;

[try {]

[MbGlobalMap map = MbGlobalMap.getGlobalMap(MapName);]

xml = (byte[])map.get(key);

[} catch (MbException e) {]

[// TODO Auto-generated catch block]

[e.printStackTrace();]

[}]

[return xml;]

[}]

[}]

You can see the Global Cache in the Activity log, within the Web
interface. Udemy 19

[\*\* \*\*]

# The THE function returns the first element of a list.

*Syntax*

THE(ListExpression)

If *ListExpression* contains one or more elements; THE
returns the first element of the list. In all other cases, it returns an empty list.

*Restrictions*

*ListExpression* must be a SELECT expression.

# DATE TIME TRANSFORMATION

IBM help "formatting and parsing date times as strings" under help in
the toolkit (Search for parsing date and time). How do we interpret date
formats?  12-09-2015 12:05pm. This is called a timestamp as it has a
time component  The first 2 digits are the day in the month. We want to
transform to :
Month in 3 letters/day in the month with 0 as padding/4 digits
year:hours,mins,seconds

```
        DECLARE INDATE CHARACTER '12-09-2015 12:05pm';

        DECLARE INDATEFORMAT CHARACTER 'd-MM-yyyy h:mma';
        --This tells us how to interpret the string in a format IIB understands
        DECLARE STDDATE TIMESTAMP CAST(INDATE AS TIMESTAMP FORMAT INDATEFORMAT);
        --We have to change it to a standard date using cast
        DECLARE OUTDATEFORMAT CHARACTER 'MMM/dd/yyyy:hh,mm,ss';
        DECLARE OUTDATE CHARACTER CAST(STDDATE AS CHARACTER FORMAT OUTDATEFORMAT);
```

He then deploys the application to the execution group.  Iibguru then
sends a blank message to the IN queue that mqinput node connects to.
Then the esql above triggers the code. The thread stops at the breakpoint
in between. Then he steps into the code, then steps over. He can see the
values of the DECLARED variables as they change.

# INTERVAL DATATYPE
How many either days, months, years (up to you) between either 2
timestamps or dates. Uncomment --- CALL CopyMessageHeaders(); so that we
can send a success message to the out queue.

```
DECLARE INDATE DATE '2009-09-17';
--- The type is DATE not CHARACTER since it is a standard date format.
--- When you run with breakpoints you will observe the variable as
--- INDATE:DATE:java.util.GregorianCalendar[time=12531....] 
--- & OUTDATE as 2009-10-17
DECLARE OUTDATE DATE INDATE+INTERVAL '1' MONTH;
```

If you want to see what day this date is:

```
DECLARE DAYFORMAT CHARACTER 'EEE';
DECLARE DAYNAME CHARACTER CAST(OUTDATE AS CHARACTER FORMAT
DAYFORMAT);
```
---DAYFORMAT is EEE, DAYNAME is Sat
---if DAYFORMAT is EEEE, DAYNAME is Saturday
---if DAYFORMAT is 'EEEE W', DAYNAME is Saturday 3 where 3 is the 3rd Saturday of that month

```
DECLARE SUBDATE CHARACTER (CURRENT_DATE-INDATE) YEAR;
---This will return an interval string not a DATE type. Hence we use CHARACTER not DATE.
SUBDATE Value is INTERVAL '8' YEAR]
DECLARE SUBDATE CHARACTER (CURRENT_DATE-INDATE) YEAR TO MONTH;
--- SUBDATE Value is INTERVAL '7-09' YEAR TO MONTH
```

If you want the differences in hours then change the INDATE
declaration from DATE type to TIMESTAMP type:

```
DECLARE INDATE TIMESTAMP '2009-09-17 13:53:25';
--- You also need to change the SUBDATE to:
DECLARE SUBDATE CHARACTER (CURRENT_TIMESTAMP-INDATE) HOUR;
```

# Opaque Parsing

In your FileNodesExamples your File Input node has an Opaque elements
search for //Opaque which means whenever the Opaque element exists
(xpath notation). It does not parse the elements inside the Opaque
parent. You will just see the child elements as a string value.

# ASBITSTREAM in ESQL

Converts payload (e.g., XML) into bitstream. Bitstream means a BLOB. A
BLOB is a string of hexadecimal characters. The benefit is that you can
insert the payload into a database. Another reason is that you may want
to cast an XML or JSON into a string.

[ ]

SET InBlob = ASBITSTREAM(InputRoot.XMLNSC,
InputRoot.Properties.Encoding, InputRoot.Properties.CodedCharSetId); --
Converts XML to BLOB so that it can be added to the Global Cache

[--This statement will convert the XML or JSON tree to string.]

SET XMLInput = COALESCE( CAST(ASBITSTREAM(InputRoot.XMLNSC CCSID 1208)
as CHAR CCSID 1208), ''); SET JSONInput = COALESCE(
CAST(ASBITSTREAM(InputRoot.JSON.Data CCSID 1208) as CHAR CCSID 1208),
'');

Every language has a particular encoding that the operating system can
understand. CCSID defines the characters that are allowed (1208 =
UTF-8). If you omit the CodedCharSetId then it will still convert fine
to a BLOB if your message is using English characters. However if the
message contains something unusual like some Cyrillic characters then
without CCSID it won't know how to handle the characters.

[]https://youtu.be/bKVmhaRqsCg?si=RB3ppXk\_-MWnaoMy Encoding 273
corresponds to Unix (Non-Intel) operating systems, for example AIX or
Solaris Spark, this is referred to as Big Endian. Encoding 546
corresponds to Linux and Windows operating systems on Intel, this is
referred to as Little Endian.

[ ]

# Aggregation

[\*\* \*\*]

[https://iteritory.com/ibm-integration-bus-iib-aggregate-nodes-sample-with-http-web-services/[]{.s15}](https://iteritory.com/ibm-integration-bus-iib-aggregate-nodes-sample-with-http-web-services/)

The following JSON message is sent to the REST API
/aggregationcustomerapi/v1/customers/.

[\*\* \*\*]

*{\
"CustomerInfoReq": \
{\
"custId": "Amit"\
},\
{\
"custId": "Shreya"\
},  \
{\
"custId": "Pete"\
}\
\
}*

[\* \*]

[\* \*]

[ ]

[The request is sent to a sub flow.]

[ ]



SET OutputLocalEnvironment.Destination.HTTP.RequestURL = reqURL ||
myref.Item[I].custId;

[                  PROPAGATE TO TERMINAL 'out' DELETE NONE;]

[                  SET I = I + 1;]

[          END WHILE;]

[          RETURN FALSE;]

[   END;]

[ ]

Within the Split compute node, the SET initializes the output
buffer.

![](IIB.fld/image009.png){style="width: 500px;height: 150px;"}

[ ]

By default, the node clears the output message buffer and reclaims the
memory when the PROPAGATE statement completes. The PROPAGATE ... DELETE
NONE will not delete the Message Assembly/Logical Tree after propagating
so that the message is available for routing to the next destination.
Use DELETE NONE if you want the downstream nodes to be able to see a
single instance of output local environment message, and exception list
trees.

[ ]

Propagation is a synchronous process. That is, the next statement
(incrementing the counter in our example) is not executed until all the
processing of the message in downstream nodes has completed. So that
means the counter 'I' will be incremented after the
[[http://localhost:7080/legacybackendservice/v1/customer/Amit]](http://localhost:7080/legacybackendservice/v1/customer/Amit)
web service response is sent to the AggregateReply.

After the while loop completes the return false command is run. This
exits the code immediately. Only now will the flow pass on to the
AggregateReply node.

[ ]

The AggregateReply node creates a folder in the combined message tree
below Root, called ComIbmAggregateReplyBody. Below this folder, the node
creates a number of subfolders using the names that you set in the
AggregateRequest nodes. These subfolders are populated with all
associated reply messages.

[ ]



![](IIB.fld/image010.png){style="width: 500px;height: 500px;"}

AggregateControl, AggregateRequest nodes (Used in Fan-Out to broadcast
the request message to multiple destinations). The request node records
the number of requests.

[ ]

AggregateReply (Used in Fan-In to collect responses) -- checks if all
responses are collected.

[ ]

'Aggregate Name' property of AggregateControl & AggregateReply nodes
should be the same. Our Aggregate Control & AggregateReply nodes have an
Aggregate name AGGR.

[https://ibmintegrationbus.wordpress.com/2019/06/14/aggregation-nodes/[]{.s15}](https://ibmintegrationbus.wordpress.com/2019/06/14/aggregation-nodes/)

[ ]

The 'Folder Name' property of the AggregateRequest node decide how the
input will be structured in Fan-Out flow. We are using RESP. RESP will
be an array for multiple responses.

[ ]

It combines the generation and concurrent fan-out of a number of
related requests with the fan-in of the corresponding replies, and
compiles those replies into a single aggregated reply message. IIB as a
product makes it easy to implement complex integration scenario with
Aggregation support.

[ ]

FanOut flows: AggregateRequest and AggregateControl (whenever we use
Aggregation control, we must use Aggregation request)

FanIn flow: AggregateReply (it will club the incoming multiple
responses) from AggregateControl node and AggregateRequest node.

[ ]

Reference:
[[https://community.ibm.com/community/user/integration/viewdocument/ibm-integration-bus-for-developers?CommunityKey=77544459-9fda-40da-ae0b-fc8c76f0ce18&tab=librarydocuments]{.s17}](https://community.ibm.com/community/user/integration/viewdocument/ibm-integration-bus-for-developers?CommunityKey=77544459-9fda-40da-ae0b-fc8c76f0ce18&tab=librarydocuments){.s16}

[ ]
