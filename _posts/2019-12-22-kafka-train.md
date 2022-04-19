# Kafka周りの開発・テストに便利なツールをdocker-composeにまとめました

最近、Kafkaを利用していたのですが、その中で便利だったツール、及び私自身がつくったツールをまとめました。
docker-composeですぐに立ち上げるようにしていますので、**とりあえずKafkaを動かしてみたい&管理してみたい**、という方におすすめです。

このdocker-composeファイルは [tarosaiba/kafka-train](https://github.com/tarosaiba/kafka-train)のリポジトリにまとめてさせてもらっています。"kafka-train"という名前はなんとなくつけました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/7910561a-4455-afe2-203c-5513abc460f9.png)


今回紹介するツールは以下です (すべてDockerコンテナとして提供されています)
* [wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)
* [wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka/)
* [tarosaiba/kafka-train-producer](https://hub.docker.com/r/tarosaiba/kafka-train-producer)
* [tarosaiba/kafka-train-consumer](https://hub.docker.com/r/tarosaiba/kafka-train-consumer)
* [tarosaiba/kafka-burrow](https://hub.docker.com/r/tarosaiba/kafka-burrow)
* [confluentinc/cp-kafka-rest:4.0.0](https://hub.docker.com/r/confluentinc/cp-kafka-rest)
* [landoop/kafka-topics-ui:0.9.3](https://github.com/lensesio/kafka-topics-ui)
* [sheepkiller/kafka-manager](https://github.com/sheepkiller/kafka-manager-docker)

# Quick Start

以下のリポジトリを利用します
[tarosaiba/kafka-train](https://github.com/tarosaiba/kafka-train)

## Start

* docker-composeで起動

```
$ docker-compose up -d
```

## Produce

* メッセージをProduceします
    - シェルスクリプトでシンプルなシーケンシャルな数字を送信します (01,02,03,04,05...)

```
$ ./send_msg.sh 5
{"id":1,"body":"Massage: 01"}
{"id":2,"body":"Massage: 02"}
{"id":3,"body":"Massage: 03"}
{"id":4,"body":"Massage: 04"}
{"id":5,"body":"Massage: 05"}
```

* 以下のように任意のメッセージをProduceすることも可能です

```
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"body":"Test massage"}' \
  localhost:1323/kafka
```

### Consume

* Consumerコンテナのログを見ると、メッセージが出力されています

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


# 各ツール紹介
## メインどころ

* [wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)
* [wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka/)

これがないと始まりません。ZookeeperとKafkaのコンテナです。(私が作ったわけではありません)
※もし、サーバのIPでKafkaにアクセスさせたいときは、`KAFKA_ADVERTISED_HOST_NAME`の使い方を気をつけてください

## Poducer & Consumer

* [tarosaiba/kafka-train-producer](https://hub.docker.com/r/tarosaiba/kafka-train-producer)
* [tarosaiba/kafka-train-consumer](https://hub.docker.com/r/tarosaiba/kafka-train-consumer)

KafkaのProducerとConsumerもセットで動かせるようにgoで簡単に作りました
Producerは、APIサーバで、`/kafka`のエンドポイントにメッセージを投げるとKafkaにメッセージをProduceします
Consumreは、メッセージをConsumeして標準出力に出力する単純なものです

Quick Startで記載しているものが、これらになります。

## Burrow

* [tarosaiba/kafka-burrow](https://hub.docker.com/r/tarosaiba/kafka-burrow)

Kafkaのモニタリングツールとしてよく使われる"Burrow"をコンテナで動かせるように簡単に作りました
RestAPIでKafkaの情報を返します。
このBurrowにより、Consumer groupのlagも取得可能です。
以下のように利用できます

```
# list topic
$ curl localhost:8888/v3/kafka/local/topic
{"error":false,"message":"topic list returned","topics":["__consumer_offsets","test_topic"],"request":{"url":"/v3/kafka/local/topic","host":"2fb74ea6b81d"}}

# list consumer
$ curl localhost:8888/v3/kafka/local/consumer
{"error":false,"message":"consumer list returned","consumers":["burrow-local","test-group"],"request":{"url":"/v3/kafka/local/consumer","host":"2fb74ea6b81d"}}
```

どのようなエンドポイントが使えるかは以下にドキュメントがあります
[Burrow-HTTP-Endpoint](https://github.com/linkedin/Burrow/wiki/HTTP-Endpoint)

## その他管理ツール
その他私が使って便利だったツール群です。(私が作ったわけではありません)

* [confluentinc/cp-kafka-rest:4.0.0](https://hub.docker.com/r/confluentinc/cp-kafka-rest)
* [landoop/kafka-topics-ui:0.9.3](https://github.com/lensesio/kafka-topics-ui)

kafka topicは、KafkaにProduceされたメッセージをGUIで見ることができます。見やすいです
kafka-restというツールを用いて、Kafkaの中のメッセージを取得するため、セットで動かす必要があります

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/5476eb2a-a709-cb87-3408-78e4b622f258.png)



* [sheepkiller/kafka-manager](https://github.com/sheepkiller/kafka-manager-docker)

kafka-managerは、Consumer groupの情報を見るのに便利です。Commit offsetとlagも見ることができます

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/108052/2a285030-e52d-74af-8c51-dd2c15a91044.png)


# まとめ

以上がKafka周りの開発・テストに便利なツールになります。ぜひ使ってみてください。

