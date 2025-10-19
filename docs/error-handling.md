<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Timeout Notification is controlled by Timeout Control](#timeout-notification-is-controlled-by-timeout-control)
- [Failure and Catch terminals](#failure-and-catch-terminals)
- [MQ Failure and Catch](#mq-failure-and-catch)
- [Try Catch node](#try-catch-node)
- [Propagation](#propagation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Timeout Notification is controlled by Timeout Control

, so long as they both have the same unique identifier. You also need
to set the operation mode of the Timeout Notification to Controlled, not
Automatic. The Timeout Control node has a Request location property
InputRoot.XMLNSC.EmpDetails.TimeoutRequest. Within the Timeout you will
have an Action, Identifier, StartDate etc. 29DEC2020-IIB
Timeout_LabelRouting_MultiQueues{.s2}\\

[ ]{.s2}

# Failure and Catch terminals

An Exception will roll back to first node (The exception will always be
in the last child of the ExceptionList) then it will check if the catch
exception is handled or not 21DEC2020-IIB FixHTTPProdCon_ESQLThrow 36
minutes. If it is then the Message and ExceptionList trees are passed to
the node at the end of the catch terminal. The ExceptionList will
contain the exception that will give you a better idea of what & where
in the flow the problem happened. If there is an exception raised while
processing downflow of the Catch then the exception will feedback then
traverse the Failure node path only if Transaction Mode is Yes.{.s2}\
If catch is not connected it will check if Failure is connected.
Then the default exception and NOT the exception we threw go to the node
at the end of the Failure terminal only if Transaction Mode is Yes. The
default exception will say you did not handle the Catch. The advantage
of the Catch terminal is that you will get an exception which will give
you a better idea of what & where in the flow the problem
happened.{.s2}

If the format of the input message is not correct, then the flow will
traverse the Failure terminal only.{.s2}

# MQ Failure and Catch

When the message that is not handled goes back to the IN queue the
message is considered a poison message as it will not allow new messages
to be 'MQGET' by the flow until the poison message is removed since it is LIFO. The
poison message can be handled by designating another queue as the
backout queue to the IN via MQ Explorer or giving the backout queue name
in the flow or setting a DLQ.21DEC2020-IIB FixHTTPProdCon_ESQLThrow 36
minutes



# Try Catch node

[]{.s2}30DEC2020-IIB RoutingTryCatchTrace. This allows us to handle the
exception further up the flow. The catch terminal will send the message
and the ExceptionList. Your terminal goes to a Compute node to handle
the exception.

# Propagation

RETURN TRUE; -- If we forget this statement, the message will still
propagate as if it was there [to the OUT terminal]{.s13}

[ ]

[PROPAGATE;]

[RETURN FALSE; --will do the same.]

[ ]

[PROPAGATE TO TERMINAL 'out' OR 0;]

[RETURN FALSE; --will do the same.]

[ ]
