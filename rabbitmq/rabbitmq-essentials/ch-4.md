# Ch 4

## Handling dead letters

- A dead letter is a message that can't be delivered, either because the intended target cannot
be accessed or because it has expired (reach their TTL).
- To implement message expiration in RabbitMQ you have 3 ways:
    1. Standard AMQP message expiration property for published messages
    2. Custom RabbitMQ extension that allows users to define a message TTL per queue
    3. Custom RabbitMQ extension that allows users to define a TTL for the queue itself

1. **Standard AMQP message expiration property for published messages**

this might sound interesting because it is a standard AMQP option; however, after reading more about how it is supported in RabbitMQ, it turns out that these messages are only discarded when the message reaches **the head** or **the beginning of a queue**.  Even if they have expired, the messages would still sit in the queue.

1. **Custom RabbitMQ extension that allows users to define a message TTL per queue**
- [https://www.rabbitmq.com/ttl.html](https://www.rabbitmq.com/ttl.html)

would be the option if you want to create an expiry date for the message in the queue.

1. **Custom RabbitMQ extension that allows users to define a TTL for the queue itself**

will delete the queue itself after the TTL.

- RabbitMQ offers the option to automatically route these dead letters to a specific exchange, a so-called **Dead letter exchange (DLX)**.
- If you want to receive messages sent to this exchange, you must bind a queue to it, consume it, and take care of received messages.
    - This queue acts as a **dead letter queue (DLQ)**, the ultimate destination for dead messages.

![Untitled](images/Untitled%202.png)

- When messages expire, they are published to the **DLX** using the *original routing key* they had when they were delivered to their original queue.
    - This behavior can be modified as RabbitMQ allows the definition of a specific routing key to be used when messages are published to the DLX.
    - But it might be more suitable if you are using the routing keys in a conventional way to expose some useful info.
    - By using the RabbitMQ extension to AMQP,
    - this can be achieved by respectively defining the **'x-message-ttl'** and **"x-dead-letter-exchange"** arguments when declaring the queue.
    - Messages published to a queue that expires after the TTL are rerouted to the exchange with the given **x-dead-letter-routing-key**.

## Policies ([Rabbitmq Policies](https://www.rabbitmq.com/parameters.html#how-policies-work))

- Now you might think we should our queue declaration code and add those arguments:

```ruby
queue1 = channel.queue('queue.name',
	 durable: true,
	 arguments: {
		 'x-message-ttl'=> 604800000, # week
		 'x-dead-letter-exchange'=> 'your-dlx',
		 'x-dead-letter-routing-key'=> 'your-queue-routing-key' # or any routing key
	 }
)
```

- The main issue is that the declaration would be changing from a queue with three arguments to a queue with no arguments (for example on the consumer side). And as we learned before in Chapter 2 a queue (or exchange) declaration is **idempotent** only if all the parameters that are used are the same. So any discrepancy in the declaration yields an exception and will be punished with an immediate channel termination.
- However, it's not the best practice as the TTL and DLX configurations are cross-cutting concerns and should be configured in a more global fashion.

RabbitMQ has a simple and elegant solution to this problem in the concept of policies.

RabbitMQ supports policies that define specific behaviors, and these policies can be applied to queues or exchanges. 

- you can apply your Policies before or after queue or exchange declarations.
- but only a single policy can apply to a queue or exchange
- Policies can only be applied through the RabbitMQ command-line tool

## Delayed messages with RabbitMQ

- The AMQP protocol doesn't have a native delayed queue feature, but one can easily be emulated by combining the message TTL function and the dead-lettering function.
    - The Delayed Message Plugin is available for RabbitMQ 3.5.3 and later versions of RabbitMQ. The Delayed Message Plugin adds a new exchange type to RabbitMQ. It is possible to delay messages routed via that exchange by adding a delay header to a message. You can read more about the plugin at [https://github.com/rabbitmq/rabbitmq-delayedmessage-exchange](https://github.com/rabbitmq/rabbitmq-delayedmessage-exchange).
- For example, you decided to publish survey request messages to a delayed queue once the driver has marked a ride as completed. All survey request messages are set to expire after a TTL of 5 minutes.
    - The routing key of the message is then changed to the same as the destination queue name.
    - This means that the survey request message will end up in the queue from which the survey request should be sent.
    - The following is an example of the code that you would use.
    - Messages are first delivered to the **DELAYED_QUEUE** called **work.later**. After 300,000 ms, messages are dead-lettered and routed to the **DESTINATION_QUEUE** called **work.now.**
    

## Making delivery mandatory

Search on the term territory in the messaging architecture 

- When a message is published on an exchange with the `mandatory` flag set to `true`, it will
be returned by RabbitMQ if the message cannot be delivered to a queue.
- A message cannot be delivered to a queue either because no queue is bound to the exchange or because none of the bound queues have a routing key that would match the routing rules of the exchange.

![Untitled](images/Untitled%203.png)

### Default exchanges in RabbitMQ

its is automatically created by RabbitMQ for each virtual host.

Each time a queue is created, it gets automatically bound to the default exchange with its queue name as the routing key. By publishing a message to the default exchange using the queue name as the routing key, the message will end up in the designated queue

<aside>
⛔ Sending messages to the default exchange is a convenient way to reach a
particular queue; however, do not overuse this pattern. It creates tight
coupling between producers and consumers because the producer
becomes aware of particular queue names.

</aside>