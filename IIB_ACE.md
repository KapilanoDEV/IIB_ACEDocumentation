## Table of Contents

- [Introduction](docs/introduction.md)

  - [Enterprise Application Integration](docs/introduction.md#enterprise-application-integration)
  - [Design, requirements gathering](docs/introduction.md#design-requirements-gathering)

- [Architecture](docs/architecture.md)

  - [Integration Nodes and servers](docs/architecture.md#integration-nodes-and-servers)
  - [MQ dependency](docs/architecture.md#mq-dependency)
  - [Application vs Integration Project](docs/architecture.md#application-vs-integration-project)
  
- [Message Transformation](docs/transformation.md)

  - [Message Transformation](docs/transformation.md#message-transformation)
  - [Transformation Extender](docs/transformation.md#transformation-extender)
  - [Transformation using XSL in a message flow](docs/transformation.md#transformation-using-xsl-in-a-message-flow)
  - [Source](docs/transformation.md#source)
  - [Target](docs/transformation.md#target)
  - [Message Enrichment](docs/transformation.md#message-enrichment)
  - [Protocol conversion](docs/transformation.md#protocol-conversion)
  - [SOA (Service Oriented Architecture)](docs/transformation.md#soa-service-oriented-architecture)
  - [Message Set, Message definition creation](docs/transformation.md#message-set-message-definition-creation)
  - [Message Set, Message definition creation for XML](docs/transformation.md#message-set-message-definition-creation-for-xml)
  - [Message modelling](docs/transformation.md#message-modelling)
  - [Mapping XML from an XSD](docs/transformation.md#mapping-xml-from-an-xsd)
  - [Mapping XML with multiple records from an XSD](docs/transformation.md#mapping-xml-with-multiple-records-from-an-xsd)

- Nodes

  - [FileInput & FileOutput nodes](docs/nodes/file-nodes.md#fileinput--fileoutput-nodes)
  - [MQEndPoint Policy](docs/nodes/mq-nodes.md#mqendpoint-policy)
  - [MQInputNode mode. How your messages will be processed](docs/nodes/mq-nodes.md#mqinputnode-mode-how-your-messages-will-be-processed)
  - [MQOutputNode mode. How your messages will be processed](docs/nodes/mq-nodes.md#mqoutputnode-mode-how-your-messages-will-be-processed)
  - [Scenario full queue MQOutputNode node](docs/nodes/mq-nodes.md#scenario-full-queue-mqoutputnode-node)
  - [MQ Output](docs/nodes/mq-nodes.md#mq-output)
  - [MQ Reply](docs/nodes/mq-nodes.md#mq-reply)
  - [Publication Node](docs/nodes/mq-nodes.md#publication-node)
  - [MQRFH2 Tree](docs/nodes/mq-nodes.md#mqrfh2-tree)

- [ESQL](docs/esql.md)

  - [Local, UDP (AKA External) & Shared Variables](docs/esql.md#local-udp-aka-external--shared-variables)
  - [Shared vs Static Library](docs/esql.md#shared-vs-static-library)
  - [ESQL native to IIB](docs/esql.md#esql-native-to-iib)
  - [MOVE NEXTSIBLING vs CARDINALITY and [i] call in a WHILE loop](docs/esql.md#move-nextsibling-vs-cardinality-and-i-call-in-a-while-loop)
  - [The MOVE statement](docs/esql.md#the-move-statement)
  - [WHILE versus FOR loop](docs/esql.md#while-versus-for-loop)
  - [Converting XMLNSC to JSON](docs/esql.md#converting-xmlnsc-to-json)
  - [Difference between DECLARE varName CHAR FIELDNAME() & DECLARE varName REFERENCE TO](docs/esql.md#difference-between-declare-varname-char-fieldname--declare-varname-reference-to)
  - [LEAVE statement](docs/esql.md#leave-statement)
  - [COALESCE](docs/esql.md#coalesce)
  - [ESQL field reference overview](docs/esql.md#esql-field-reference-overview)
  - [Shared variable and ATOMIC block](docs/esql.md#shared-variable-and-atomic-block)
  - [Global Cache](docs/esql.md#global-cache)
  - [The THE function returns the first element of a list](docs/esql.md#the-the-function-returns-the-first-element-of-a-list)
  - [DATE TIME TRANSFORMATION](docs/esql.md#date-time-transformation)
  - [INTERVAL DATATYPE](docs/esql.md#interval-datatype)
  - [Opaque Parsing](docs/esql.md#opaque-parsing)
  - [ASBITSTREAM in ESQL](docs/esql.md#asbitstream-in-esql)
  - [Aggregation](docs/esql.md#aggregation)

- [Java](docs/java.md)
  - [Java and Java Compute Node](docs/java.md#java-and-java-compute-node)
  - [JCN code to access a message set](docs/java.md#jcn-code-to-access-a-message-set)

- [Routing](docs/routing.md)
  - [Filter node](docs/routing.md#filter-node)
  - [RouteToLabel node](docs/routing.md#routetolabel-node)

- [Security](docs/security.md)
  - [HTTPS webservice](docs/security.md#https-webservice)
  - [Set up HTTPS web service](docs/security.md#set-up-https-web-service)

- [Deployment](docs/deployment.md)
  - [Compile BAR inline](docs/deployment.md#compile-bar-inline)
  - [BAR override](docs/deployment.md#bar-override)

- [Error Handling](docs/error-handling.md)
  - [Timeout Notification is controlled by Timeout Control](docs/error-handling.md#timeout-notification-is-controlled-by-timeout-control)
  - [Failure and Catch terminals](docs/error-handling.md#failure-and-catch-terminals)
  -   [MQ Failure and Catch](docs/error-handling.md#mq-failure-and-catch)
  - [Try Catch node](docs/error-handling.md#try-catch-node)
  - [Propagation](docs/error-handling.md#propagation)

- [Monitoring](docs/monitoring.md)
  - [Event Monitoring using node properties](docs/monitoring.md#event-monitoring-using-node-properties)
  - [Event Monitoring using monitoring profile XML file](docs/monitoring.md#event-monitoring-using-monitoring-profile-xml-file)

- [Troubleshooting](docs/troubleshooting.md)  
  - [Validation settings for input type nodes](docs/troubleshooting.md#validation-settings-for-input-type-nodes)

- [Web Services](docs/web-service.md)
  - [Web Services](docs/web-service.md#web-services)
  - [SOAP Gateway mode](docs/web-service.md#soap-gateway-mode)
  - [Exposing SOAP webservice. WSDL creation](docs/web-service.md#exposing-soap-webservice-wsdl-creation)
  - [REST](docs/web-service.md#rest)

- [SAP](docs/sap.md)
  - [SAP](docs/sap.md#sap)
  - [How to use IBM App Connect with SAP (via RFC)](docs/sap.md#how-to-use-ibm-app-connect-with-sap-via-rfc)





