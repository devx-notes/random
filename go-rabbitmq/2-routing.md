# Routing

## 1. Declaring

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

## 2. Binding

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