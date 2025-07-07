# Work Queues

## 1. Connection

A single connection is intended to last for the full lifetime of the process.


```go
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
if err != nil {
    panic(err)
}
defer conn.Close()
```

Channels are created per logical task (publisher, consumer group, etc.) and are not thread‑safe.

```go
ch, err := conn.Channel()
if err != nil {
    panic(err)
}
defer ch.Close()
```

## 2. Publishing Messages

```go
err = ch.PublishWithContext(ctx,
  "",           // exchange
  q.Name,       // routing key
  false,        // mandatory
  false,
  amqp.Publishing {
    DeliveryMode: amqp.Persistent,
    ContentType:  "text/plain",
    Body:         []byte(body),
})
```

## 3. Consuming Messages

The Qos (Quality of Service) method controls how many messages the broker will deliver to a consumer before it must acknowledge them.

- A value of 0 is treated as infinite, allowing any number of unacknowledged messages.
- Two prefetch limits should be enforced independently of each other; consumers will only receive new messages when neither limit on unacknowledged messages has been reached.

```go
err = ch.Qos(
    1,     // prefetch count: the maximum number of unacknowledged messages
    0,     // prefetch size: the maximum total size (in bytes) of unacknowledged messages
    false, // global: scope of the prefetch setting, "false" for consumer, "true" for all consumer on the channel
)
```

Every consumer has an identifier that is used by client libraries to determine what handler to invoke for a given delivery. Their names vary from protocol to protocol. Consumer tags are also used to cancel consumers.

```go
msgs, err := ch.Consume(
    q.Name, // queue
    "",     // consumer tag
    true,   // auto-ack
    false,  // exclusive
    false,  // no-local (not supported by RabbitMQ)
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

