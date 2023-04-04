<h3 align="center">Kafka S3 Sink POC</h3>

#### Process

* Create DigitalOcean droplet (Ubuntu 22.10 x64)
* Install Kafka

```bash
sudo apt update && sudo apt upgrade
sudo apt install default-jdk 
wget https://archive.apache.org/dist/kafka/3.3.1/kafka_2.13-3.3.1.tgz
tar xzf kafka_2.13-3.3.1.tgz
sudo mv kafka_2.13-3.3.1.tgz /usr/local/kafka
```
```bash
sudo vim /etc/systemd/system/zookeeper.service 
```
```
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
```bash
sudo vim /etc/systemd/system/kafka.service 
```
```
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload 
sudo systemctl start zookeeper 
sudo systemctl start kafka
sudo systemctl status zookeeper 
sudo systemctl status kafka 
```

<br>

* Create topic

```bash
# from /usr/local/kafka
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic chip-orders

# consume topic w/ key
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orders --property print.key=true --property key.separator="-"
```

<br>

* Generate fake data using my project - [Fake Chipotle Streaming](https://github.com/kadekillary/fake-chipotle-streaming)

>**Note** had to make a few edits

```
// added strconv lib
// updated topic to chip-orders

// line 50 - before - dind't like `-` in key
time.Now().UTC().Format("2006-01-02 15:04:05")
// after
strconv.Format(time.Now().UnixMilli(), 10)
```

```bash
git clone https://github.com/kadekillary/fake-chipotle-streaming.git

sudo apt install golang-go

# be in /src dir
go mod download

# start sending data to topic
go run *.go
```

https://user-images.githubusercontent.com/25046261/229938277-afda6335-2100-4154-ba19-9d9bc885aab5.mov

<br> 

#### Create Kafka S3 Sink

```bash
wget http://client.hub.confluent.io/confluent-hub-client-latest.tar.gz
tar -xvf confluent-hub-client-latest.tar.gz
export PATH="$HOME/bin:$PATH"
cd /usr/local/share
mkdir -p kafka/plugins
confluent-hub install confluentinc/kafka-connect-s3:10.4.2 --component-dir /usr/local/share/kafka/plugins --worker-configs /usr/local/kafka/config/connect-distributed.properties
```

Create a new configuration file for the S3 connector, e.g., `s3-sink.properties` - location: `/usr/local/kafka/config/`. Configure the S3 connector with the required properties - [Docs](https://docs.confluent.io/kafka-connectors/s3-sink/current/overview.html#amazon-s3-sink-connector-for-cp):

```
name=s3-sink
connector.class=io.confluent.connect.s3.S3SinkConnector
tasks.max=1

topics=chip-orders
s3.region=us-west-1
s3.bucket.name=chip-orders

aws.access.key.id=your_aws_access_key
aws.secret.key=your_aws_secret_key

format.class=io.confluent.connect.s3.format.json.JsonFormat
storage.class=io.confluent.connect.s3.storage.S3Storage
flush.size=1000
rotate.interval.ms=3600000
```

* Start S3 Connector

Edit `/config/connect-standalone.properties` since not following schema/payload

```
key.converter.schemas.enable=false
value.converter.schemas.enable=false
```

Start the S3 connector using the Kafka Connect CLI:

```bash
$ ./bin/connect-standalone.sh ./config/connect-standalone.properties ./config/s3-sink.properties
```

Validate

<img width="808" alt="Screen Shot 2023-04-04 at 3 36 33 PM" src="https://user-images.githubusercontent.com/25046261/229937539-07d27300-750d-4444-979a-c79f31c323ba.png">

*Note: Make sure you have the necessary permissions to access the S3 bucket and Kafka topic.*
