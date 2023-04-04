<h3 align="center">Kafka S3 Sink POC</h3>

#### Process

* Create DigitalOcean droplet
* Install [Kafka](https://tecadmin.net/how-to-install-apache-kafka-on-ubuntu-22-04/)
```
wget https://archive.apache.org/dist/kafka/3.2.0/kafka_2.13-3.2.0.tgz
```
* Create topic

```bash
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partition 1 --topic orders
```

* Generate fake data using my project - [Fake Chipotle Streaming](https://github.com/kadekillary/fake-chipotle-streaming)
* Create Kafka S3 Sink from notes in my Notion article

```bash
wget http://client.hub.confluent.io/confluent-hub-client-latest.tar.gz
tar -xvf confluent-hub-client-latest.tar.gz
export PATH="$HOME/bin:$PATH"
confluent-hub install confluentinc/kafka-connect-s3:10.4.2 --component-dir /usr/local/kafka/libs --worker-configs /usr/local/kafka/config/connect-console-sink.properties
```

Create a new configuration file for the S3 connector, e.g., `s3-sink.properties`. Configure the S3 connector with the required properties - [Docs](https://docs.confluent.io/kafka-connectors/s3-sink/current/overview.html#amazon-s3-sink-connector-for-cp):

```
name=s3-sink
connector.class=io.confluent.connect.s3.S3SinkConnector
tasks.max=1

topics=my-kafka-topic
s3.region=us-west-2
s3.bucket.name=my-s3-bucket

aws.access.key.id=your_aws_access_key
aws.secret.key=your_aws_secret_key

format.class=io.confluent.connect.s3.format.avro.AvroFormat
storage.class=io.confluent.connect.s3.storage.S3Storage
flush.size=1000
rotate.interval.ms=3600000
```

Replace the placeholders with your actual Kafka topic, S3 region, S3 bucket, and AWS credentials. You can also customize `flush.size` and `rotate.interval.ms` based on your requirements.

**Start the S3 connector:**

Start the S3 connector using the Kafka Connect CLI:

```bash
$ ./bin/connect-standalone.sh ./config/connect-standalone.properties ./config/s3-sink.properties
```

The above command assumes you are in the Kafka installation directory and that the configuration files are in the `config/` directory.

**Verify the data in AWS S3:**

After starting the S3 connector, it will begin consuming messages from the specified Kafka topic and writing them to the specified S3 bucket. Navigate to the AWS S3 console and verify that the data is being written to the S3 bucket.

*Note: Make sure you have the necessary permissions to access the S3 bucket and Kafka topic.*
