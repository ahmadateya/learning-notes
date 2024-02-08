# Ch 6: clustering

# RabbitMQ On production

RabbitMQ provides the features needed to deal with potential crashes and other catastrophes right out of the box

- In OurEdu we need multiple nodes to achieve high availability, but a small machines is enough

<aside>
💡 The way a cluster of two RabbitMQ brokers is created is really similar to what is typically
done when making a *relational database highly available*

</aside>

### To transform your node into a cluster you should:

- add a node and configure it in the dashboard.
- modify the connection in your apps to rotate over the nodes if one fails, connect to another **(Not a good thing!)**
- enabling a way to spread the data to the other node(s), the options available
    - **classic mirrored queues** and **quorum queues**

## Partition handling strategies

When a distributed system consists of multiple nodes, issues such as **split-brains** and network partitions can arise. 

A **split-brain** occurs when a portion of the network becomes isolated from another portion, leading to network partitions.

To address these challenges, a partition handling strategy needs to be defined. In RabbitMQ, this is done through the **`cluster_partition_handling`** parameter in the configuration file.

There are two specific strategies:

1. **Pause-Minority Strategy:**
    - This strategy involves terminating nodes in the minority partition when a split-brain occurs.
    - It is a common default way to resolve split-brains in distributed networks.
2. **Pause-If-All-Down Strategy:**
    - This strategy only pauses a node if none are reachable.
        - Not recommended as it can lead to large discrepancies between the data in each partition.

When nodes become available again after a partition, two options are provided for reconnecting the network:

- Ignore the other partition.
- Autoheal the cluster.

In the case of the pause-minority strategy, partitions automatically reconnect when available.

---

## RabbitMQ queues Types

Queues in RabbitMQ can be `durable` or `transient`. 

Classic mirrored queues are recommended for transient message handling, while quorum queues are a good alternative for durable queues.

Another queue type, `lazy queues`, writes the contents to disk as early as possible for both durable and transient messages.

<aside>
🚧 Didnt complete this section as it contains a lot of info that is useful only when you decide to cluster your nodes by hand.

</aside>

---

## Handling log processing

<aside>
🚧 Didnt complete this section as it contains a lot of info that is useful only when you decide to collect the logs from clustered nodes by hand.

</aside>