---
title: "Azure Service Bus Transport Type"
date: 2021-01-24T21:39:59-05:00
draft: false
description: "All things Azure and AWS"
---


# Azure Service Bus Transport Type
By default, the Azure Service Bus will use a TransportType of Advanced Message Queuing Protocol (AMQP), which uses the assigned port number is 5672 or 5671 for AMQPS (TLS/SSL encrypted AMQP). These ports can be blocked in some enterprises.

It is not well documented, but there are ways to configure the Azure Service Bus clients to use WebSockets, on port 443, which is never blocked, because this is also the assigned port for SSL or HTTPS.

## .Net Service Bus client
For a .Net service bus client pushing a message to a Service Bus, you can use the ServiceBusClientOptions object. as follows. ServiceBusClientOptions is an argument you can use when constructing a ServiceBusClient object. Since the ServiceBusClient object is the one that initiates a connection to a ServiceBus, it is easy to establish the TransportType at initial connection to use AmqpWebSockets.
```
ServiceBusClientOptions options = new ServiceBusClientOptions { TransportType = ServiceBusTransportType.AmqpWebSockets }; 
```
If you don’t set the TransportType, you are likely to see the following exception:
```
Azure.Messaging.ServiceBus.ServiceBusException: 'An existing connection was forcibly closed by the remote host. ErrorCode: ConnectionReset (ServiceCommunicationProblem)'
Inner Exception:
SocketException: An existing connection was forcibly closed by the remote host.
```
## Azure Function
For an Azure Function, the Service Bus itself (or the Azure intermediary that triggers your function) is the one that initiates a connection, not your Azure Function code. To accomplish that connection to use AmqpWebSockets, you have to modify the ServiceBus ConnectionString that the Azure Function uses, to include the following parameter:
```
            ;TransportType=AmqpWebSockets
```
### For example:
```
Endpoint=sb://croinsdevncusmessagingsb.servicebus.windows.net/;SharedAccessKeyName=FunctionAppAccessKey;SharedAccessKey=xyzExampleKey123;TransportType=AmqpWebSockets
```
The first part of the connection string was from the Azure Service Bus, and the part in red was added later to change the TransportType so that the Ip Port of 443 is used for the trigger.
 