<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Compile BAR inline](#compile-bar-inline)
- [BAR override](#bar-override)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Compile BAR inline

[15 minutes 04JAN2021-IIBFileNodeBARMQReplyTryCatchFlowOrder]

To include message flows as compiled message flow (.cmf) files, and to
include ESQL code directly in the .cmf file of each message flow that
references an ESQL file, select Compile and in-line resources. By
default, when you add a message flow to a BAR file, it is added as a
.msgflow file. By default, each ESQL file that is referenced by one or
more of your message flows is deployed as an individual resource, and
can be accessed by multiple .msgflow files. If any of the flows that you
add to your BAR file contain a subflow that is defined in a .msgflow
file, you must select Compile and in-line resources.

[ ]

# BAR override

[Notes from Udemy Section 12.]

Under Independent resources -> GeneratedBarFiles you can copy the bar
file to a folder or run mqsireadbar
(04JAN2021-IIBFileNodeBARMQReplyTryCatchFlowOrder 20 minutes).

[ ]

After extracting the file you can copy lines from the
META-INF/broker.xml and create different property files for different
environments.

[ ]

[The line from the broker.xml below:]

[ ]

\<ConfigurableProperty override="HELLO"
uri="ExternalVariable#EXTVAR"/>

[ ]

[Can be copied to a new file called DEV.properties]

[ ]

[ExternalVariable#EXTVAR = diffvalue]

[ ]

Then you can run mqsiapplybaroverride -b \<path to
desktop>/MAP_PRO.generated.bar -p DEV.properties -k MAP_PRO

[ ]

Then go to \<path to desktop>/MAP_PRO.generated.bar, copy it, go to
the Toolkit, right click on the MAP_PRO application then, paste it. This
results in a new BAR folder under the application that contains the new
generated BAR file. Click on the new BAR and you will see the new
properties in the RHS.

[ ]

You can then run this command to deploy the BAR with the new
properties.

[ ]

$> mqsideploy BROKERNAME -e EG_NAME -a \<path to
desktop>/MAP_PRO.generated.bar

[ ]
