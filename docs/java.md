<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Java and Java Compute Node](#java-and-java-compute-node)
- [JCN code to access a message set](#jcn-code-to-access-a-message-set)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Java and Java Compute Node

[JAN082021-IIB ExtJavaCall_DbNode_MQReplyPubSub](https://drive.google.com/file/d/1-IJJvmRgPg6FxfbKL6MKDrQzAwPy2h9c/view?usp=share_link)

```
--Creating Procedure For Calling External_Java_File(REGSTR.java),method RECORDS
CREATE PROCEDURE EXJAVACALL(IN PRO1C INTEGER,IN PRO2C INTEGER,IN VNAME CHARACTER) RETURNS CHARACTER LANGUAGE JAVA
EXTERNAL NAME "com.mss.REGSTR.RECORDS";
```

IIB 10 uses IBM java which includes oracle java's 1.7 features. IBM
Integration Bus v10 ships with Java 8 in IIB 10.0.0.11 and later fix
packs on all supported distributed platforms except Solaris and HP.
However, the IIB Toolkit is unable to compile Java source code
containing Java 8 language constructs such as Functional Interfaces,
default and static methods in Interfaces and Lambda Expressions. If you
try to deploy a bar file which contains or references Java 8 compiled
class files to an IIB node running 10.0.0.10 or earlier then you may see
one of the following errors: JavaCompute node with an onInitialize
method BIP4157E: The user-defined node 'Java Compute' could not be
deployed. Details: java.lang.UnsupportedClassVersionError: JVMCFRE003
bad major version class=Java18Test, offset=6.

![JCN Wizard](images/JCNWizard.png)

MbJavaComputeNode is the superclass of all the classes used in JCN.

[JAN182021-IIB JCN_MessSet 5:08](https://drive.google.com/file/d/1IzB8MWQEh2HPMKkJkW2gxkHXKuraxiGI/view?usp=share_link)
The Filtering message class in JCN wizard is not aimed for transformation. It can't construct the new message. It is used to check the data without changing it like a filter node, then
apply filtering rules.
JCN template for Java Architecture for XML Binding (JAXB) class relates
to marshalling and unmarshalling. Java classes are generated from an
XSD. You can create new XML data using the constructor's classes and
methods. You can also read newly created XML data.

When you have finished the template wizard you will see extends
MbJavaComputeNode. This superclass is an abstract class. It has abstract
methods and concrete methods. Abstract methods that are already
implemented can be called in our JCN. With concrete methods are not
implemented so we need to implement the declaration. The evaluate method
is a concrete method that we need to implement. The evaluate method is
akin to the Main method in ESQL.

```
public void evaluate(MbMessageAssembly inAssembly) throws MbException {
  // evaluate is akin to the CREATE FUNCTION Main() RETURNS BOOLEAN

 
// MbMessage root1 = inAssembly.getMessage(); // access Input tree
MbElement root = inAssembly.getMessage().getRootElement(); // InputRoot
MbElement Body = inAssembly.getMessage().getRootElement().getLastChild().getLastChild(); // Data under XMLNSkC

String jlKt=""; 

// Outputtree pointer
MbElement xmlnsc = outAssembly.getMessage().getRootElement().getLastChild();

// INPUT
MbElement empDetails=root.getLastChild().getLastChild().getFirstChild();
// MbElement empDetails=root.getLastChild()
String SName=empDetails.getFirstChild().getValueAsString();
String SmpID=empDetails.getFirstChild().getNextSibling().getValueAsString();
String SmpAge=empDetails.getFirstChild().getNextSibling().getNextSibling().getValueAsString();
```

The MbMessageAssembly is capable of holding the entire logical tree.
The logical tree consists of 4 trees:

1. message tree (properties, MQMD, XMLNSC)
1. LocalEnvironment
1. Environment
1. ExceptionList

![Message tree in debug](IIB.fld/Variables.png)
![How java code traverse tree](images/ac30330a.gif)
![How code relates to tree](IIB.fld/javaCNtree.png)

# JCN code to access a message set

In google drive "[2021-01-27] IIBFixJCNMulti.mp4" you can access MSET
using XPath.

| |
|----|
| [MbElement root = inAssembly.getMessage().getRootElement().getLastChild().getLastChild();] |

| |
|---------------------------------------------------------------|
| MbElement emp1[]= root.getAllElementsByPath("\*"); |

| |
|-----------------------|
| [              ] |

| |
|----|
| [MbElement Root=outAssembly.getMessage().getRootElement().getFirstChild();] |

| |
|----|
| [Root.getFirstElementByPath("./MessageSet").setValue("JCN_CSV_MessageSet");] |

| |
|----|
| [Root.getFirstElementByPath("./MessageType").setValue("{}:SDET");] |

| |
|----|
| [Root.getFirstElementByPath("./MessageFormat").setValue("Text_CSV");] |

| |
|----------|
| [ ] |

[\*\* \*\*]
