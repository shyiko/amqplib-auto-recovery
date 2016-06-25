# amqplib-auto-recovery

Automatic connection recovery for [amqplib](https://github.com/squaremo/amqp.node). Node.js 6+.

## Installation

```sh
npm install amqplib-auto-recovery --save
```

## Usage

Wrap amqp client with auto recovery decorator (like shown below).
API stays absolutely the same as before *. The only difference is that `amqp.connect(...)`'s callback
will be executed each time (re)connection is attempted (exponential backoff is turned on by default).

```javascript
const amqp = require('amqplib/callback_api');
const withAutoRecovery = require('amqplib-auto-recovery');

withAutoRecovery(amqp, {
  // onError: (err) => { console.log(err.message) },
  // isErrorUnrecoverable: (err) => false
  // for more see src/amqplib-auto-recovery.js
}).connect("amqp://localhost", (err, con) => {
    if (err) {
      console.error(`Failed to connect (${err.message})`);
      return
    }
    console.info("Connection established");
    con.createChannel((err, ch) => {
      if (err) {
        console.error(`Failed to create a channel (${err.message})`);
        return
      }
      ch.prefetch(1, false);
      ch.assertQueue(consumer.queue, {durable: true});
      ch.consume(consumer.queue, (msg) => {
        // handle msg   
      });
    });
  });
```

\* with exception to `closed` property (added to connection/channel), which
might come in handy in situations like:

```javascript
      // ...
      ch.consume(consumer.queue, (msg) => {
        // simulating a long-running task
        setTimeout(() => {
          if (ch.closed) {
            console.log("Channel had been closed before task was completed");
            return
          }
          ch.ack(msg); // throws IllegalOperationError if channel is closed
          // note that this is a standard amqplib behavior and not something
          // introduced by amqplib-auto-recovery
        }, 5000);
      });
```

## License

[MIT License](https://github.com/shyiko/amqplib-auto-recovery/blob/master/mit.license)
