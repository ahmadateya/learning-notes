# Ch 5

## WebSockets in RabbitMQ

- RabbitMQ is a multi-protocol message broker
    - it uses the **Single TextOriented Message Protocol (STOMP)**
    - you can use it for example to send GPS data if you are tracking a device

## Reply-to queues and RPC

- you can use Rabbitmq queues to communicate 2 ways
- for example if you have a message that the consumer needs to return something, you can use the `reply-to` message property to send the routing key you need for the other queue.

### **You can use any tye of queues but in practice, the following two approaches are used:**

 **1. Create a short-lived queue for each request-response interaction.** 

This approach uses an exclusive, auto-deleted, nondurable, server-side named queue created by the client with the following benefits:

- No other consumer can get messages from it since it is exclusive.
- It can be auto-deleted; once the reply has been consumed, there is no longer a use for it.
- No need for it to be durable; request-response interactions are not meant to be long-lived.
- The server generates a unique name, which relieves the client from having to figure out a unique naming scheme.

 **2. Use a permanent reply-to queue specific to the client** 

This approach uses a nonexclusive, non-automatically deleted, nondurable, client-side named queue with the following benefits:

- No need for it to be durable, for the same reason explained previously.
- No need for it to be exclusive—a different consumer will be used for each request-response interaction.
- A permanent queue is more efficient than using short-lived queues with each request since creation and deletion are expensive operations.

<aside>
⛔ The difficulty in using a permanent queue is in correlating responses with requests. This is done through the `CorrelationId` message property, carried from the request message to the response message. This property allows the client to identify the correct request to process.

</aside>

<aside>
💡 RabbitMQ client libraries offer primitives that simplify responses
correlated with requests.

</aside>

## Headers Exchange

The headers exchange allows the routing of messages based on their headers, which are custom key-value pairs stored in the message properties. 

The custom key-value pairs guide messages to the correct queue or queues. With this approach, the message and its routing information are all self-contained, remain consistent, and are therefore easier to inspect as a whole.