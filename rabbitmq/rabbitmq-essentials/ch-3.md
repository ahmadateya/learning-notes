# Ch 3

### Prefetch count

- In the consumer, you can set the prefetch count to fetch some messages at once
- if the prefetch count is too small it will affect the Rabbitmq performance, since the platform is waiting for permission to send more messages.
- For example, if a consumer has the prefetch set to one
    - this means RabbitMQ will not send the next message until after the delivery, processing,
    and acknowledgment of the message is complete.
    - If the message processing time is 5ms, the message time cose will be 125ms (60ms + 5ms + 60ms)

![Untitled](images/Untitled%201.png)

- s
- RabbitMQ supports **channel-level**, **message-based** prefetch counts, not connection or byte size-based prefetching.
- A **low prefetch** value is recommended in situations where there are **many consumers and a
short processing time**. If the prefetch value is set too low, the consumers will be idle much
of the time, waiting for messages to arrive.
- On the other hand, if the prefetch value is too high, one consumer may be very busy while the others are idle
    - One typical mistake is to allow unlimited prefetch where one client receives all the messages, leading to high memory consumption and crashes, which cause all messages to be re-delivered.
    
    ---
    

### Fanout Exchange

- To send a message to all queues, you could create a topic exchange and make every client subscribe to it.
- But the cleaner and simpler solution is to use a **Fanout exchange**
- The routing keys in the fanout exchange do not really matter, because the fanout exchange does not care about them.
- Examples of using fanout exchange :
    - Scoreboard or leaderboard updates from sports news to mobile clients, or other global events
    - Broadcasting various state and configuration updates in distributed systems