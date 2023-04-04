<h3 align="center">Kafka S3 Sink POC</h3>

#### Process

* Create DigitalOcean droplet
* Install [Kafka](https://tecadmin.net/how-to-install-apache-kafka-on-ubuntu-22-04/)
* Create topic

```bash
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partition 1 --topic orders
```

* Genrate fake data using my project - [Fake Chipotle Streaming](https://github.com/kadekillary/fake-chipotle-streaming)
* Create Kafka S3 Sink from notes in my Notion article
