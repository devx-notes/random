# RabbitMQ Queues

References: [rabbitmq/go](https://www.rabbitmq.com/tutorials/tutorial-one-go)

RabbitMQ is a message broker: it accepts and forwards messages.

## 1. Introduction

RabbitMQ, and messaging in general, uses some jargon

### Message Lifecycle

When a message enters RabbitMQ it passes through three distinct phases

| Phase     | Internal state | Description                                                                                                 |
| --------- | -------------- | ----------------------------------------------------------------------------------------------------------- |
| Queued    | Ready          | The message resides on disk or in memory, assigned to a queue but not yet delivered.                        |
| In‑flight | Unacknowledged | The message has been sent to a consumer, yet the broker has not received an explicit acknowledgment for it. |
| Completed | Acked / Nacked | The consumer has responded, the broker then removes or re‑queues the message accordingly.                   |

Consumer acknowledgments (delivery‑side reliability)  

| Method                     | Effect                                                                    | Typical use        |
| -------------------------- | ------------------------------------------------------------------------- | ------------------ |
| `basic.ack`                | Confirms successful processing; message is deleted.                       | Normal completion. |
| `basic.nack requeue=true`  | Signals failure; message is re‑queued for another attempt.                | Transient errors.  |
| `basic.nack requeue=false` | Routes the message to a dead‑letter exchange (if configured) or drops it. | Poison messages.   |
| `basic.reject`             | Single‑message variant of NACK; identical semantics.                      | Edge cases.        |

With autoAck=true the broker discards the message immediately after sending it, eliminating the unacknowledged phase but forfeiting at‑least‑once guarantees.

> [!CAUTION]
> Publisher confirms are an independent protocol feature

| Direction         | Mechanism                | Guarantees                                                              |
| ----------------- | ------------------------ | ----------------------------------------------------------------------- |
| Producer → Broker | Publisher Confirm        | Message has been persisted and routed to at least one queue.            |
| Broker → Consumer | Delivery Acknowledgment  | Message has been processed (or intentionally rejected) by the consumer. |


### Channels

Some applications need multiple logical connections to the broker. However, it is undesirable to keep many TCP connections open at the same time because doing so consumes system resources and makes it more difficult to configure firewalls. 

- AMQP 0-9-1 connections are multiplexed with channels.
- Channels can be thought of as lightweight connections that share a single TCP connection.
- Much like connections, channels are meant to be long lived (channel pool). 
- The client cannot be configured to allow for more channels than the server configured maximum

### Exchange 

In AMQP 0-9-1, exchanges are the entities where publishers publish messages that are then routed to a set of queues or streams.

- Exchanges routes all messages that flow through them to one or more queues, streams, or other exchanges.
- Every exchange belongs to one virtual host (logical groups of entities)

## 2. Work Queues

## 3. Pub/Sub

### Other Solutions

#### Apache Kafka

#### Apache ActiveMQ

#### Redis
