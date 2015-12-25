[![Build Status](https://travis-ci.org/oleksiyk/kafka.png)](https://travis-ci.org/oleksiyk/kafka)

# no-kafka

no-kafka is [Apache Kafka](https://kafka.apache.org) 0.9 client for Node.js with new unified consumer API support. No Zookeeper connection required. 

All methods will return a [promise](https://github.com/petkaantonov/bluebird)

* download and install Kafka
* create your test topic: 
```shell
kafka-topics.sh --zookeeper 127.0.0.1:2181 --create --topic kafka-test-topic --partitions 3 --replication-factor 1
```

## Producer

Example:

```javascript
var Kafka = require('no-kafka');
var producer = new Kafka.Producer();

return producer.init().then(function(){
  return producer.send({
      topic: 'kafka-test-topic',
      partition: 0,
      message: {
          value: 'Hello!'
      }
  });
})
.then(function (result) {
  /*
  { ok: [ { topic: 'kafka-test-topic', partition: 0, offset: 353 } ],
  errors: [] }
  */
});
```

### Producer options:
* requiredAcks - require acknoledgments for produce request. If it is 0 the server will not send any response.  If it is 1 (default), the server will wait the data is written to the local log before sending a response. If it is -1 the server will block until the message is committed by all in sync replicas before sending a response. For any number > 1 the server will block waiting for this number of acknowledgements to occur (but the server will never wait for more acknowledgements than there are in-sync replicas).
* timeout - timeout in ms for produce request
* clientId - ID of this client, defaults to 'no-kafka-client'
* connectionString - comma delimited list of initial brokers list, defaults to '127.0.0.1:9092'

## Simple Consumer

Manually specify topic, partition and offset when subscribing. Suitable for simple use cases.

Example:

```javascript
var consumer = new Kafka.SimpleConsumer();

consumer.on('data', function (messageSet, topic, partition) {
    messageSet.forEach(function (m) {
        console.log(topic, partition, m.offset, m.message.value.toString('utf8'));
    });
});

return consumer.init().then(function () {
    return Promise.all([
        consumer.subscribe('kafka-test-topic', 0),
        consumer.subscribe('kafka-test-topic', 1)
    ]);
});
```

Subscribe (or change subscription) to specific offset and limit maximum received MessageSet size:
```javascript
consumer.subscribe('kafka-test-topic', 0, {offset: 20, maxBytes: 30}
```

Subscribe to latest or earliest offsets in the topic/parition:
```javascript
consumer.subscribe('kafka-test-topic', 0, {time: Kafka.LATEST_OFFSET}
consumer.subscribe('kafka-test-topic', 0, {time: Kafka.EARLIEST_OFFSET}
```

Commit offset(s) (V0, Kafka saves these commits to Zookeeper)
```javascript
consumer.commitOffset([
  {
      topic: 'kafka-test-topic',
      partition: 0,
      offset: 1
  },
  {
      topic: 'kafka-test-topic',
      partition: 1,
      offset: 2
  }
])
```

Fetch commited offset(s)
```javascript
consumer.fetchOffset([
  {
      topic: 'kafka-test-topic',
      partition: 0
  },
  {
      topic: 'kafka-test-topic',
      partition: 1
  }
]).then(function (result) {
/*
[ { topicName: 'kafka-test-topic',
    partitions:
     [ { partition: 0,
         offset: 1,
         metadata: null,
         error: null },
       { partition: 1,
         offset: 2,
         metadata: null,
         error: null },
      ] 
    },
]
*/
});
```

### Simple Consumer options
* groupId - group ID for comitting and fetching offsets. Defaults to 'no-kafka-group-v0'
* timeout - timeout for fetch requests, defaults to 100ms
* idleTimeout - timeout between fetch calls, defaults to 1000ms
* minBytes - minimum number of bytes to wait from Kafka before returning the fetch call, defaults to 1 byte
* maxBytes - maximum size of messages in a fetch response
* clientId - ID of this client, defaults to 'no-kafka-client'
* connectionString - comma delimited list of initial brokers list, defaults to '127.0.0.1:9092'

## Group Consumer (new unified consumer API)

Specify an assignment strategy (or use no-kafka built-in consistent assignment strategy) and subscribe by specifying only topics. Elected group leader will automatically assign partitions between all group members. 

Example:

```javascript

```