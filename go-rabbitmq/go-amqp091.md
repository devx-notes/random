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
- Whenever a named exchange other than `""` is used, at least one `QueueBind` explicit binding is essential; otherwise messages simply vanish.

### Binding

A binding refers to how exchanges are connected to queues or to other exchanges.

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

This tells RabbitMQ to deliver messages published to `exchangeName` with this `routingKey` into `queueName`

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

This tells RabbitMQ to route messages sent to `sourceExchange` with `routingKey` to `destinationExchange` as if they were published directly.

### 2.1. Fire-and-forget

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

### 2.2. Publisher confirms

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

## References

- [GitHub | RabbitMQ Go Tutorials](https://github.com/rabbitmq/rabbitmq-tutorials/tree/main/go)