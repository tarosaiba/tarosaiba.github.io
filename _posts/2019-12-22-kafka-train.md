# Useful tools for development and testing around Kafka in docker-compose

I have recently been using Kafka. Kafka-Train is a list of tools that I have found useful, as well as tools that I have created myself.
I wrote docker-compose file to get it up and running immediately, so if you want to **run & manage Kafka for now**, this is a good choice.

This docker-compose file has been put together in the [tarosaiba/kafka-train](https://github.com/tarosaiba/kafka-train) repository." The name "kafka-train" is just a random name.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/7910561a-4455-afe2-203c-5513abc460f9.png)


The following tools are introduced in this article (all are provided as Docker containers)
* [wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)
* [wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka/)
* [tarosaiba/kafka-train-producer](https://hub.docker.com/r/tarosaiba/kafka-train-producer)
* [tarosaiba/kafka-train-consumer](https://hub.docker.com/r/tarosaiba/kafka-train-consumer)
* [tarosaiba/kafka-burrow](https://hub.docker.com/r/tarosaiba/kafka-burrow)
* [confluentinc/cp-kafka-rest:4.0.0](https://hub.docker.com/r/confluentinc/cp-kafka-rest)
* [landoop/kafka-topics-ui:0.9.3](https://github.com/lensesio/kafka-topics-ui)
* [sheepkiller/kafka-manager](https://github.com/sheepkiller/kafka-manager-docker)

# Quick Start

Use the following repositories
[tarosaiba/kafka-train](https://github.com/tarosaiba/kafka-train)

## Start

* Launch with docker-compose

```
$ docker-compose up -d
```

## Produce

* Produce message
    - Send simple sequential numbers in shell script (01,02,03,04,05...)

```
$ ./send_msg.sh 5
{"id":1,"body":"Massage: 01"}
{"id":2,"body":"Massage: 02"}
{"id":3,"body":"Massage: 03"}
{"id":4,"body":"Massage: 04"}
{"id":5,"body":"Massage: 05"}
```

* It is also possible to produce a message which you want as follows

```
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"body":"Test massage"}' \
  localhost:1323/kafka
```

### Consume

* The log in the Consumer container outputs the message which we produced

```
$ docker logs -f kafka-train_consumer_1
Kafka Broker -  kafka:9092
Kafka topic -  test_topic
subscribed to topic  test_topic
waiting for event...
Message Massage: 01
waiting for event...
Message Massage: 02
waiting for event...
Message Massage: 03
waiting for event...
Message Massage: 04
waiting for event...
Message Massage: 05
waiting for event...
```


# Introduction of each tool
## Main tools

* [wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)
* [wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka/)

You can't start without these. They are containers for Zookeeper and Kafka.
If you want Kafka to access the server IP, be careful how you use `KAFKA_ADVERTISED_HOST_NAME`!

## Poducer & Consumer

* [tarosaiba/kafka-train-producer](https://hub.docker.com/r/tarosaiba/kafka-train-producer)
* [tarosaiba/kafka-train-consumer](https://hub.docker.com/r/tarosaiba/kafka-train-consumer)


I made it easy in go to run Kafka Producer and Consumer as a set.
Producer is an API server that produces messages to Kafka when you throw messages to the `/kafka` endpoint.
Consumre is a simple way to consume messages and output them to standard output.

## Burrow

* [tarosaiba/kafka-burrow](https://hub.docker.com/r/tarosaiba/kafka-burrow)

I made a simple Docker image of "Burrow", which is often used as a Kafka monitoring tool, to run in a container.
Returns Kafka information via RestAPI. This Burrow also allows you to get the Consumer group's lag. It can be used as follows

```
# list topic
$ curl localhost:8888/v3/kafka/local/topic
{"error":false,"message":"topic list returned","topics":["__consumer_offsets","test_topic"],"request":{"url":"/v3/kafka/local/topic","host":"2fb74ea6b81d"}}

# list consumer
$ curl localhost:8888/v3/kafka/local/consumer
{"error":false,"message":"consumer list returned","consumers":["burrow-local","test-group"],"request":{"url":"/v3/kafka/local/consumer","host":"2fb74ea6b81d"}}
```

Here is the documentation on what endpoints are available
[Burrow-HTTP-Endpoint](https://github.com/linkedin/Burrow/wiki/HTTP-Endpoint)

## Other management tools
These are tools that I have used and found useful. (I did not create these tools.)

* [confluentinc/cp-kafka-rest:4.0.0](https://hub.docker.com/r/confluentinc/cp-kafka-rest)
* [landoop/kafka-topics-ui:0.9.3](https://github.com/lensesio/kafka-topics-ui)


kafka topic allows you to view messages Produced in Kafka in a GUI. It is easy to see
A tool called kafka-rest is used to retrieve messages in Kafka, so it is necessary to run with.

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/5476eb2a-a709-cb87-3408-78e4b622f258.png)


* [sheepkiller/kafka-manager](https://github.com/sheepkiller/kafka-manager-docker)

kafka-manager is useful for viewing Consumer group information, including Commit offset and lag

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/2a285030-e52d-74af-8c51-dd2c15a91044.png)


# Conclusion

These are useful tools for development and testing around Kafka. I encourage you to use them!
