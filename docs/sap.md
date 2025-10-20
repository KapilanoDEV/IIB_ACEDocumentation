<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [SAP](#sap)
- [How to use IBM App Connect with SAP (via RFC)](#how-to-use-ibm-app-connect-with-sap-via-rfc)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# SAP

[Connecting to Enterprise Information
Systems](https://www.ibm.com/docs/en/app-connect/12.0?topic=applications-connecting-enterprise-information-systems)

WebSphere Broker Adapters Transport is a service that connects to
EIS (such as SAP). To use the transport, your message flow must contain
one or more WebSphere Adapter nodes, for example the SAPRequest node
(needs an outbound adapter component, which is used by the message flow
to invoke a service in the EIS).

With the adapter for SAP software you can create integrated processes
that include the exchange of information with an SAP server, without
special coding. The adapter creates a standard interface to the
applications and data on the SAP server so that the developer of the
application component does not have to understand the lower-level
details (the implementation of the application or the data structures)
on the SAP server. The adapter complies with the Java Connector
Architecture (JCA) 1.5, which standardizes the way in which application
components, app servers, and EIS, interact with each other.

SAPRequest node -> sends a request to SAP which will then call up to
the BAPI (Business Application Programming Interface). BAPI is a
function that SAP provides, in order to get access to the data. The
nodes also need XML Schema Definitions (XSD) to ensure that ACE messages
are propagated to and from nodes reflect the logical structure of the
data in the EIS.

# How to use IBM App Connect with SAP (via RFC)

[Connecting to SAP (via
RFC)](https://www.ibm.com/docs/en/app-connect/12.0?topic=hga-sap-via-rfc#index__connect__title__1)

When you double click say a SAPRequest node an Adapter component
selection window appears. Then click on import where you create your own
adapter. To connect to SAP you need a connector file. Adapter for SAP
Software is packaged and delivered as two RAR files, and the one you use
depends on whether the invoked SAP function supports transactional
behavior.

If the targeted function (for example, BAPI) supports transactions, use
the CWYAP_SAPAdapter_Tx.rar adapter because it supports local
transaction behavior and, as such, can participate in the transaction
managed by the WebSphere Application Server Transaction Manager.

If the targeted function (for example, BAPI) does not support
transactions, use the CWYAP_SAPAdapter.rar adapter because it indicates
to the WebSphere Application Server Transaction Manager that the
interaction performed with the SAP system cannot participate in and
follow transactional semantics.

You will need to contact your administrator for the SAP client files (Go
to the SAP Support Portal and download the SAP Java™ Connector and the
SAP Java IDoc Class Library, which together contain the following files:
**sapjco3.jar, libsapjco3.so, sapidoc3.jar**) as well as for the
connection details for how to connect to SAP (hostname or IP address of
the SAP system, two-digit system number of the instance that you want to
access in the SAP system; for example, 00, three-digit number of the
client that you want to access within the SAP instance; for example,
110, ISO two-letter case-insensitive code that identifies the language
setting of the SAP system; for example, en or EN, code page, username,
password) & to find out which BAPIs you will need to be invoking.
WebSphere Adapter for SAP Software connects to SAP systems running on
SAP Web application servers. The adapter supports Advanced Event
Processing (AEP) and Application Link Enabling (ALE) for inbound
processing, and the Business Application Programming Interface (BAPI),
AEP, ALE, and Query Interface for SAP Systems (QISS) for outbound
processing. You set up the adapter to perform outbound and inbound
processing by using the Adapter Connection wizard to generate business
objects based on the services it discovers on the SAP server.

For outbound processing, the adapter client invokes the adapter
operation to create, update, or delete data on the SAP server or to
retrieve data from the SAP server.

For inbound processing, an event that occurs on the SAP server is sent
from the SAP server to the adapter. The ALE inbound and BAPI inbound
interfaces start event listeners that detect the events. Conversely, the
Advanced event processing interface polls the SAP server for events. The
adapter then delivers the event to an endpoint, which is an application
or other consumer of the event from the SAP server.

You configure the adapter to perform outbound and inbound processing by
using the Adapter Connection wizard to create a library that includes
the interface to the SAP application as well as business objects based
on the functions or tables that it discovers on the SAP server. In the
wizard we can add a filter to the RFC search results. For example find
objects with this pattern BAPI_CUSTOMER\*. We create the adapter in the
library because the library will remain constant even if we regenerate
the pattern (https://www.youtube.com/watch?v=4r6gNuE-6is 5:50). We name
the library say ContactBook_ProcedureHandlers. It will contain CRUD
subflows. Finally in the properties tab at the bottom of the page set
the default setting to GetDetail2 as this is the read procedure. Under
the ContactBook_ProcedureHandlers library is a schema definitions
folder. We open the data model for GetDetail2. This XSD is quite complex
but WMB simplifies this by using the transformation functions. Drag a
Compute node between Input and ReadCustomer in the sub-flow. This
Compute node will build the request message to be sent to the SAPRequest
node
https://www.ibm.com/docs/en/app-connect/12.0?topic=scenarios-migrating-sap-integration-flow.

When you create a SAPRequest node has a Response Message Parsing
properties. The Message domain property specifies the domain that is
used to parse the response message. The DataObject domain is the default
domain when parsing messages that are produced by the SAPRequest node.
**DataObject domain. You cannot specify a different domain**. The node
automatically detects the message type.

A DataObject is typically used with the Clipboard and in drag-and-drop
operations. The DataObject class provides the recommended implementation
of the IDataObject interface. It is suggested that you use the
DataObject class rather than implementing IDataObject yourself. Multiple
pieces of data in different formats can be stored in a DataObject. Data
is retrieved from a DataObject by its associated format. Because the
target application might not be known, you can increase the likelihood
that the data will be in the appropriate format for an application by
placing the data in a DataObject in multiple formats.

However, when passing data to the SAPRequest node (for example, by using
an MQInput node), the use of a different domain can improve performance.
For example, use the XMLNSC parser with the MQInput node to parse XML
messages.

[← Back to Main page](../IIB_ACE.md)