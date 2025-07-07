# Using RabbitMQ in Go

In AMQP, clients publish Exchanges, not to Queues. The routing between producers and consumer queues is via Bindings. These bindings form the logical topology of the broker.

## rabbitmq/amqp091-go

An AMQP 0-9-1 Go client maintained by the RabbitMQ team. Originally `streadway/amqp`

```sh
go get github.com/rabbitmq/amqp091-go
```

In this library, a message sent from publisher is called a `Publishing` and a message received to a consumer is called a `Delivery`.

## 1. Connection

- A single connection is intended to last for the full lifetime of the process.
- Channels are created per logical task (publisher, consumer group, etc.) and are not thread‑safe.

```go
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
if err != nil {
    panic(err)
}
defer conn.Close()

ch, err := conn.Channel()
if err != nil {
    panic(err)
}
defer ch.Close()
```

## 2. Publishing Messages

Declare a queue to send messsage to

```go
q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)
if err != nil {
    panic(err)
}
```

> [!IMPORTANT]
> In AMQP 0‑9‑1 the topology‑declaration steps `ExchangeDeclare`, `QueueDeclare`, and `QueueBind` are not strictly mandatory in every program.

- Re‑declaring queue with identical attributes is harmless but attempting to re‑declare with different attributes triggers channel‑level error 406. For this reason, `QueueDeclare` is optional but appears in most production examples. 
- If the producer and the consumer agree to use a pre‑declared exchange like `""` or `amq.topic`, `ExchangeDeclare` can be skipped .

### Binding

A binding refers to how exchanges are connected to queues or to other exchanges. This one tells RabbitMQ to deliver messages published to `exchangeName` with this `routingKey` into `queueName`

```go
// tells the exchange how to route messages
ch.QueueBind(
    queueName,    // the queue to bind
    routingKey,   // key to match on publish
    exchangeName, // the exchange to bind to
    false,
    nil,
)
```

This one tells RabbitMQ to route messages sent to `sourceExchange` with `routingKey` to `destinationExchange` as if they were published directly.

```go
// links one exchange to another
ch.ExchangeBind(
    destinationExchange,    // the exchange to bind to
    routingKey,             // key to match on publish
    sourceExchange,         // the exchange to bind to
    false,
    nil,
)
```

Whenever a named exchange other than `""` is used, at least one `QueueBind` or `ExchangeBind` explicit binding is essential; otherwise messages simply vanish.

- Adding QueueBind to the publish routine is not required and is usually avoided
- Responsibility for bindings is normally assigned to the consumer side or to a dedicated bootstrap step, it helps keeping the publish path lean while still guaranteeing correct routing through proper topology management elsewhere.

```go
err := ch.ExchangeDeclare(
    "exchange-name", // name
    "fanout",        // exchange-type: fanout, topic, direct
    true,            // durable
    false,           // auto-delete
    false,           // internal
    false,           // no-wait
    nil,             // arguments
)
if err != nil {
    return err
}

err = consumeCh.QueueBind(
		"queue-name",
		"routing-key",
		"exchange-name",
		false,
		nil,
	)
	if err != nil {
		return err
	}
```

## 3. Consuming Messages

The Qos (Quality of Service) method controls how many messages the broker will deliver to a consumer before it must acknowledge them.

```go
err = ch.Qos(
    1,     // prefetch count: the maximum number of unacknowledged messages
    0,     // prefetch size: the maximum total size (in bytes) of unacknowledged messages
    false, // global: scope of the prefetch setting, "false" for consumer, "true" for all consumer on the channel
)
if err != nil {
    panic(err)
}
```

Every consumer has an identifier that is used by client libraries to determine what handler to invoke for a given delivery. Their names vary from protocol to protocol. Consumer tags are also used to cancel consumers.

```go
msgs, err := ch.Consume(
    q.Name, // queue
    "",     // consumer tag
    true,   // auto-ack
    false,  // exclusive
    false,  // no-local
    false,  // no-wait
    nil,    // args
)
if err != nil {
    panic(err)
}

go func() {
    for d := range msgs {
        log.Printf("Received a message: %s", d.Body)
    }
}()
```

## 4. Consumer Acknowledgements and Publisher Confirms 

When registering a consumer, applications can choose one of two delivery modes

- Automatic (deliveries require no acknowledgement, a.k.a. "fire and forget")
- Manual (deliveries require client acknowledgement)

### 4.1. Fire-and-forget

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

body := "Hello World!"
err = ch.PublishWithContext(ctx,
    "",     // exchange
    q.Name, // routing key
    false,  // mandatory
    false,  // immediate
    amqp.Publishing{
        ContentType: "text/plain",
        Body:        []byte(body),
    })
```

Messages are immediately marked as acknowledged — no retry on failure or crash.

```go
deliveries, err := ch.Consume(
    q.Name, 
    "",
    true, // autoAck = true
    false, 
    false, 
    false, 
    nil,
)
if err != nil {
    return err
}
```

### 4.2. Client Acknowledgements

Publisher confirms and consumer delivery acknowledgements are very similar features that solve similar problems in different contexts. However, they are entirely orthogonal and unaware of each other.

- Publisher confirms cover publisher communication with RabbitMQ 
- Consumer acknowledgements cover RabbitMQ communication with consumers. The goal is to confirm to a RabbitMQ node that a given delivery was successfully received and processed successfully, so the delivered message can be marked for future deletion.

#### 📢 Publisher Confirms

Confirm mode introduces a round‑trip but removes the need for transactions and is the canonical reliability mechanism. An exclusive goroutine per publisher is recommended.

```go
if err := ch.Confirm(false); err != nil {    
    panic(err)
}
ackCh := ch.NotifyPublish(make(chan amqp.Confirmation, 1))

err = ch.Publish(
    // ...
)
if err != nil {
    panic(err)
}

confirm := <-ackCh
if !confirm.Ack {
    // message was nacked or the channel closed – trigger retry
}

```

#### 🖥️ Consumer Acknowledgements

The AMQP 0‑9‑1 protocol delivers each message with a delivery‑tag that identifies the position of that delivery on its channel.
After a message has been processed the client must decide one of three actions: `Ack`, `Nack` or `Reject` 

```go
deliveries, err := ch.Consume(q, "", false, false, false, false, nil)
if err != nil {
    return err
}

for {
    select {
    case d, ok := <-deliveries:
        if !ok { return amqp.ErrClosed }      // channel closed by broker
        if err := handle(d.Body); err != nil {
            _ = d.Nack(false, true)          // requeue on error
            continue
        }
        _ = d.Ack(false)                     // single positive ack
    case <-ctx.Done():
        _ = ch.Cancel("", false)             // stop deliveries cleanly
        return nil
    }
}
```


## References

- [GitHub | RabbitMQ Go Tutorials](https://github.com/rabbitmq/rabbitmq-tutorials/tree/main/go)
- [Acknowledger Interface](https://github.com/rabbitmq/amqp091-go/blob/v1.10.0/delivery.go#L19)