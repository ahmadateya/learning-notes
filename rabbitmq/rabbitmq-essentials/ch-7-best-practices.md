# Ch 7: Best Practices

## How to avoid losing messages

Here are some best practice recommendations for how to not lose messages in RabbitMQ:

- Keep `queues` short
- Use at least three nodes in the RabbitMQ cluster, and the **quorum queue** type to spread messages to different nodes.
- If all messages must be processed, declare a queue as `durable` and set the message delivery mode to `persistent`.
    - Queues, exchanges, and messages need to be able to handle any restarts, crashes, or hardware failures that may occur.

### Some clarifications regarding message handling in RabbitMQ:

- Understanding the trade-offs that come with persistence is essential when designing a durable system architecture.
    - **Lazy queues**, though using transient messages, have a similar effect on performance.
- Using transient messages with durable queues creates speed without losing
configuration but may result in message loss.

<aside>
❓ What if all these best practices are followed and messages are still in jeopardy of being lost? **`DLX`**

</aside>

---

### Using a dead letter exchange

- It is recommended for applications that often get hit by spikes of messages to set a queue `max-length`. The queue max-length helps keep the queue short by discarding messages from the head of the queue. The max-length can be set to *a number of messages*, or *a set number of bytes.* **(a weird advice, where will the messages go if I discard them from the queue!***)*

---

### Handling acknowledgments and confirms

- It is important to remember that the application that consumes important messages should not `ack` until it is handled
- Confirms can also have an impact on system performance, but they are required if the publisher must process messages at least once.

### Limiting message size

The number of messages sent per second is a much larger concern than the size of the messages themselves. However, sending large messages is not a best practice, and neither is sending too small messages since AMQP adds a small packet overhead to all messages sent.

**Examine messages to see whether they can be split and sent to different queues, as follows:**

- Split iterable data into chunks and send a small chunk per message.
- Store large files in a distributed store, such as Hadoop or networked attached storage.
- Split the architecture into more modular components with a queue per component.
- Offload appropriate metadata to a key-value store.

While sending large messages can be avoided, bandwidth, architecture, and fail-over limits are a consideration. The size of the message will depend on the application *but should be as small as possible.*

---

### Using consumers and prefetching

- it is important to remember that prefetching is only effective when all consumers are busy
- A prefetch value that is
    - `too low` keeps the consumers idle as they wait for messages to arrive, which in turn will slow down the broker's ability to handle requests.
    - Setting the prefetch value `too high` keeps one consumer busy while the rest remain idle
- If processing time is low and the network is stable, then the prefetch value can be increased.
In this case, the prefetch value can be determined easily by dividing the total round trip time by the processing time.

---

## Keeping queues and brokers clean

- A clean broker is an efficient broker

### 1. Setting the TTL for messages or the max-length on queues

- Setting the `TTL` allows messages to be removed from the queue after a certain time. If specified, these messages enter the dead letter exchange. This saves more messages and even handles potential issues without losing data.

### 2. Auto-deleting unused queues

In addition to keeping queues from becoming overly large, queues can be dropped based on use. 

There are three ways to delete an unused queue automatically: 

1. Set an expiration policy for the queue using the `x-expires` property on the declaration, keeping queues alive only for a number of non-zero milliseconds when unused.
2. Set the `auto-delete` queue property to true on the declaration. This means the
queue will be dropped after the following scenarios:
    - The initial connection is made.
    - The last consumer shuts down.
    - The channel/connection is closed or the queue has lost the TCP connection with the server.
3. Set the `exclusive` property to true on queue declaration so that the structure belongs to the declaring connection and is deleted when the connection closes.

---

## Routing best practices

- direct exchanges are the fastest to use
- ***Design a system with routing in mind***

### Networking over connections and channels

A large number of connections and channels:

- Consumes a lot of memory, causing it to run out of memory and crash.
- Can also negatively impact the RabbitMQ management interface due to the large number of
performance metrics being processed.
- To avoid this, configure each application to create an *extremely small number of connections*
    - Instead of using multiple connections, establish a channel for each thread.
    - As some clients don't make channels `thread-safe`, it is best not to share channels between threads
- Repeatedly opening and closing connections and channels is also detrimental to system performance

---

<aside>
🚧 The rest of the best practices are very advanced to go with, so you should search about them if you need something there

</aside>