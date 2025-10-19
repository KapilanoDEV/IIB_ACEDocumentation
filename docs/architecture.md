<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Integration Nodes and servers](#integration-nodes-and-servers)
- [MQ dependency](#mq-dependency)
- [Application vs Integration Project](#application-vs-integration-project)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Integration Nodes and servers

There is no option to create the node from the toolkit. You can run

```
mqsicreatebroker ACE12NODE (*optional* -i ace12user -a ace12user) -q QMGR
```

The integration node is represented by the node.conf.yaml that can be
overriden by a node.conf.yaml in the overrides directory. If you use
commands to modify the integration node, the changes are saved in the
integration node's override node.conf.yaml file.
Within the node.conf.yaml you can set the administration REST API port
for the ACE web user interface and the ACE Toolkit, specify the host,
enable authentication, enable administration security and specify an
authorization mode. You can also modify properties to enable the
integration node listener, to configure the MQTT server, and to specify
a default queue manager.

```
      RestAdminListener:
        port: 4414   # Set the Admin REST API Port for ACE Web UI and Toolkit. Defaults to 4414

  # Note the Admin REST API will be insecure without the following being set
        host: 'localhost'  # Set the hostname otherwise we bind to the unspecified address
        basicAuth: true
        adminSecurity: active 
        authMode: file
      NodeHttpListener:
        startListener: true                   # Enables the Integration Node listener
        HTTPConnector:
          ListenerPort: 7080           # Set non-zero to set a specific port, defaults to 7080
      MQTTServer:
          #enabled: true                            # Enables the MQTT server
          #port: 11883                              # Sets the port for the MQTT server

      defaultQueueManager: 'AMITQMGR'     # Set non-empty string to specify a default queue manager
```

When you create integration servers that are owned by the integration
node, a default server.conf.yaml configuration file is created for each
of the integration servers, and they are stored in the file system in
subdirectories below the integration node directory. Any properties that
you set for the integration node, in the node.conf.yaml file, are
inherited by the integration servers that it owns. If you have a message
flow that contains an MQEndpoint policy that connects to queue manager
ACEQM & you deploy it to an independent Integration Server (without a
queue manager specified in it's server.conf.yaml) you will see the
message:

> BIP1361E Message flow node 'MQ Input',
> 'com.udemy.ace12_26.MQEndpointTest#FCMComposite_1_1' in Message flow
> 'com.udemy.ace12_26.MQEndpointTest',
> 'com.udemy.ace12_26.MQEndpointTest' requires Policy 'Default' of
> type 'MQEndpoint' which is not deployed.

If you deploy with a QMGR in the server.conf.yaml then deployment
completes without problems. If you stop the IServer, remove the
defaultQueueManager from the conf file, then try to start it again you
will see:

> BIP1361E Message flow node 'MQ Input',
> 'com.udemy.ace12_26.MQEndpointTest#FCMComposite_1_1' in Message flow
> 'com.udemy.ace12_26.MQEndpointTest',
> 'com.udemy.ace12_26.MQEndpointTest' requires Policy 'Default' of
> type 'MQEndpoint' which is not deployed. The identified resource
> requires a policy which has not been deployed. If this message is
> reported while the message flow is being deployed then this will not
> cause the deploy operation to fail however the message flow will be
> unable to start until the missing policy is deployed. Deploy the
> missing policy, or update the message flow to remove the dependancy on
> the policy.

When you create a managed integration server for an integration node,
server-specific settings are created for it in its own server.conf.yaml
file. The server-specific file is located at:
$MQSI_WORKPATH/components/"Node name"/servers/"Node owned Server
name"/server.conf.yaml.

However, you can change any of the integration server properties
by modifying them in the appropriate server.conf.yaml file. For more
information about configuring integration servers that are managed by an
integration node, see Configuring an integration server by modifying the
server.conf.yaml file.
The location of the server.conf.yaml file that you need to modify
depends on whether you are configuring an independent integration server
or an integration server that is managed by an integration node.

Modify the properties that you want to change in the file. For example:
In the RestAdminListener section, set a value for the Port property to
be used by the REST administration port, which is the primary method of
communicating with the integration server. (Default value of 7600.) In
the ResourceManagers / HTTPConnector section, set a value for the
ListenerPort so that you can send messages to a flow that is using a
HTTPInput node. (Default value of 7800.) In the ResourceManagers / JVM
section, set a value for the jvmDebugPort so that you can use the Flow
Debugger. For example, set this property to 9997.

```
    RestAdminListener:
      port: 7600

      ResourceManagers:
      HTTPConnector:
        ListenerPort: 7800
      JVM:
        jvmDebugPort: 9997
```

# MQ dependency

IIB only needs MQ if you are using MQ nodes. Before IIB v9 it was
called WMB where MQ is a prerequisite. The QMGR contains Integration
Node configuration. IIB v10 MQ is not a prerequisite.

# Application vs Integration Project

Both are a Container, but the Application provide isolation since you
can use the resources it contains. This is needed for Microservices
where the MS is contained.

[← Back to Main page](../IIB_ACE.md)