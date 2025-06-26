# RabbitMQ Queues

References: [rabbitmq/go](https://www.rabbitmq.com/tutorials/tutorial-one-go)

RabbitMQ is a message broker: it accepts and forwards messages.

## 1. Introduction

RabbitMQ, and messaging in general, uses some jargon

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
