---
title: RabbitMQ
---

## Communication Patterns

### Point to Point (Direct Exchange)

There is exactly one sender and one receiver. In case there are multiple
receivers that are applicable for the purpose of receiving the message, only one
of them succeeds. The sender does not receive a response in this method.

### Publish Subscribe (Fanout Exchange)

There is one sender and multiple receivers. It is fire-and-forget, where the
sender does not await for a response.

### Request Response (2x Exchanges & 2x Queues)

There is one sender and one receiver that sends a response to the sender of the
message.

## Virtual Host (vhost)

The logical units that divide RabbitMQ server components (such as exchanges,
queues, and users) into separate groups for better administration and access
control. Each AMQP client connection is bound to a concrete virtual host.

## Exchanges

These are the RabbitMQ server endpoints to which the clients will connect and
send messages. Each endpoint is identified by a unique key.

There is multiple exchange types:

- Direct: This delivers a message based on a routing key that is provided in the
  message header (bindings should already be defined between the direct exchange
  and the queue). There is a pre-created direct exchange with the name
  .amq.direct. A specialized type of a direct exchange called default exchange
  with the empty string as the exchange name is also pre-created in the message
  broker. It has the special property where the binding key that is specified by
  the client should match the name of the queue to which a message is routed.

- Fanout: This delivers a message to all the queues that are bound to the
  exchange; it can be used to establish a broadcast mechanism for the delivery
  of messages to the queues. There is a pre-created fanout exchange with name
  .amq.fanout.

- Topic: This delivers the message to queues based on a routing filter specified
  between the topic exchange and queues; it can be used to establish a multicast
  mechanism for the delivery of messages. There is a pre-created topic exchange
  with the name .amq.topic.

- Headers: This can be used to deliver messages to queues based on other message
  header attributes (and not the routing key). There are two pre-created headers
  exchanges with names .amq.headers and .amq.match.

## Routing keys

- (*) One word: `foo.*.qux` matches: `foo.bar.qux`
- (#) Multiple words: `#.qux` matches `abc.bar.qux`

## Queues

These are the RabbitMQ server components that buffer messages coming from one or
more exchanges and send them to the corresponding message receivers. The
messages in a queue can also be offloaded to a persistent storage (such queues
are also called durable queues) that provides a higher degree of reliability in
case of a failed messaging server; once the server is running again, the
messages from persistent storage are placed back in the corresponding queues for
transfer to recipients. Each queue is identified by a unique key.

Receivers can either subscribe to a queue in order to receive messages (also
called _push-style_ communication) or request messages on demand from a queue
(also called _pull-style_ communication).

### Queue properties

- Name
- Durable (the messages, queues and exchanges will survive a broker restart)
- Exclusive (used by only one connection and the queue will be deleted when that
  connection closes)
- Auto-delete (queue that has had at least one consumer is deleted when last
  consumer unsubscribes)

### Dead letter queue

Queue containing messages that can not be sent to their destinations for some
reason. Such as:

- Message that is sent to a queue that does not exist.
- Queue length limit exceeded.
- Message length limit exceeded.
- Message is rejected by another queue exchange.
- Message reaches a threshold read counter number, because it is not consumed.
  Sometimes this is called a _back out queue_.

This can be done using policies or by setting the `x-dead-letter-exchange`
argument when declaring a queue.

### Message delivery modes

_Persistent_ messages are stored on disk and are guaranteed to be delivered,
while _non-persistent_ messages are not stored on disk and may not be delivered.

### Message delivery guarantees

This allows the sender to specify a delivery guarantee, such as at-least-once or
exactly-once. This ensures that the message is delivered to the receiver at
least once, or exactly once, depending on the guarantee specified.

### Message acknowledgement mode

_Auto-acknowledge_ mode will automatically acknowledge messages as soon as they
are received, while _manual-acknowledge_ mode requires the consumer to manually
acknowledge messages.

### Message expiration

This allows the sender to specify a time limit for the message to be delivered.
If the message is not delivered within the specified time limit, it will be
discarded.

### Mandatory messages

If not consumed, or sent to any queue, it will be returned to producer.

## Bindings

These are the logical link between exchanges and queues. Each binding is a rule
that specifies how the exchanges should route messages to queues. A binding may
have a routing key that can be used by clients in order to specify the routing
semantics of a message.
