<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Message Routing](#message-routing)
  - [Filter node](#filter-node)
  - [RouteToLabel node](#routetolabel-node)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Message Routing

Application A -XML-> ESB -> B (XML)

Application A -JSON-> ESB -> C (JSON)

Suppose you have a Route node preceding the RCD node. Within it you can
set XPaths. These are used to extract the header information. In the
Postman request you can set the content type to XML or JSON. Then the
XPath dictates which node to propagate to. It might send the message to
the RCD where the domain is set.

## Filter node

Use the Filter node with an ESQL statement to determine the next node
to which the message is sent by this node. Do not use the ESQL code that
you develop for use in a Filter node in any other type of node.

## RouteToLabel node

Use the RouteToLabel node after a Compute node or a JavaCompute node
for complex routing. Define a list of destinations in a Compute or
JavaCompute node that are acted on by the RouteToLabel node. The
RouteToLabel node interrogates the destinations and passes the message
on to the corresponding Label node.