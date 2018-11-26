---
layout: post
title:  "Add changelog and EventStreams to a legacy APP using Kafka and kafka Connect"
date:   2018-11-24 00:00:00
categories: Kafka Changelog Mysql
tags:
 - Kafka
 - Changelog
 - Mysql
 - draft
---


Add changelog and EventStreams to a legacy APP using Kafka and kafka Connect

### Mission: 

1. Emmit events everytime data changes in an existing application, without touching the original codebase.
2. Use those events to create a live changelog of the app

### High level milestones:

1. Get a legacy app based on MYSQL database
2. Setup Kafka, Kafka connect and debezium
3. Configure Kafka Connect to detect changes on specific tables 

### Extra mile: 

1. Configure Kafka Connect to detect changes on aggregated data (Joined tables)
2. Create a custom transformation to modify the generated output

### Let´s start


#### STEP #1 - Get a legacy app based on MYSQL database

The application should follow some rules:

* Have a MYSQL database
* Tables should have an `last_updated_at` timestamp column or an incremental revision id that automatically updates on every row save
* MYSQL binlog should be enabled

if you don't have any app like that, this is as simple drupal example to load with `docker-compose`

a. create a file named `docker-compose.yml` with the following content inside:

	version: '3.1'
	services:
	  drupal:
	    image: drupal
	    ports:
	      - 8080:80
	    volumes:
	      - /var/www/html/modules
	      - /var/www/html/profiles
	      - /var/www/html/themes
	      - /var/www/html/sites
	    restart: always
	  mysql:
	    image: mysql
	    ports:
	      - 23306:3306
	    command: --default-authentication-plugin=mysql_native_password
	    restart: always
	    environment:
	      MYSQL_ROOT_PASSWORD: root
	      MYSQL_ALLOW_EMPTY_PASSWORD: "true"
	      MYSQL_DATABASE: drupal_db



      
b. Run `docker-compose up -d` on the same folder as the file lives

c. When the drupal container is up, you can access the setup page at: [http://localhost:8080](http://localhost:8080) and complete the setup:

	profile: "Demo"
	db_type: "MySQL"
	db_name: "drupal_db"
	user: "root"
	pass: "root"
	host:  "mysql"

d. __After finish this experiment__, we can cleanup all the containers we just created by running: `docker-compose stop && docker-compose rm -f`
 
e. Validate database is accessible from the host machine (I'm using [mycli](https://www.mycli.net/) but you can use any other client) :

	# connect
	mycli mysql://root@127.0.0.1:23306/drupal_db
	
	# check the tables are there
	SHOW TABLES;  

	# check the binlog is enabled
	SHOW VARIABLES LIKE 'log_bin';

	#should be like:
	+-----------------+---------+
	| Variable_name   | Value   |
	|-----------------+---------|
	| log_bin         | ON      |
	+-----------------+---------+
	1 row in set
	Time: 0.019s


 
_Once finished we will get this:_

![Example App based on Drupal](/files/kafka-changelog/example-app.png)

This is a good moment to get up  :raised_hands:, stretch your legs :walking:, and grab another cup of :coffee: because we're just about to start the fun part!  

#### STEP #2 - Setup Kafka, Kafka connect and debezium

For our purposes we will use the __Confluent Open Source__ Kafka quick start bundle  [https://www.confluent.io/download/](https://www.confluent.io/download/)

1. Download and unzip the confluent package (in my case `confluent-oss-5.0.1-2.11.zip`)
2. Add mysql support by downloading jdbc driver for [mysql](https://dev.mysql.com/downloads/connector/j/5.1.html) `mysql-connector-java-5.1.47` and adding it to `share/java/kafka-connect-jdbc`
3. Add Debezium plugin (this will be usefull to also apply changes to the streamed data). Download the latest from [https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/](https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/) (in my case `debezium-connector-mysql-0.9.0.Beta1-plugin.tar.gz`) unzip it and move the extracted folder to `share/java/`



#### STEP #3 - Configure Kafka Connect to detect changes on specific tables 

In this step we are going to use 4 terminal sessions: "zookeper", "kafka server", "Kafka avro console consumer", and the "confluent CLI" 

1. Start Zookepper: `./bin/zookeeper-server-start ./etc/kafka/zookeeper.properties`
2. Then in another window, start Kafka server: `./bin/kafka-server-start ./etc/kafka/server.properties`
3. And in a third one, start the Kafka avro console consumer: `./bin/kafka-console-consumer  --bootstrap-server localhost:9092 --topic test --from-beginning`
4. In the 4th terminal let's start by testing kafka is working with a simple test: `./bin/kafka-console-producer --broker-list localhost:9092 --topic test` 
At this point you can type anything and every time you hit enter the message will be sent to the `test` topic and will appear in the `console consumer`
![Testing Kafka producer and consumer are working](/files/kafka-changelog/test_kafka.gif)

Now stop the producer (Ctrl + C) and create a new file `/tmp/kafka-connect-jdbc-source.json`

This file is going to be the configuration for kafka-connect.

Here is my version adapted from [https://gist.github.com/rmoff/85b6f26a592203e70b9366999680ac05](https://gist.github.com/rmoff/85b6f26a592203e70b9366999680ac05) to make it fit with our example drupal database:

<script src="https://gist.github.com/PabloGancharov/4705322cb5b538cfbfa8cf493ed3ecac.js"></script>

As you can see I white-listed the table `node__body` and I configured the log observation strategy as `incrementing` because Drupal uses integers to represent that a row has changed. Other modes are `timestamp` or `bulk` more on that on [https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/source_config_options.html#mode](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/source_config_options.html#mode)

OK, so next we will load that configuration into Connect:

	# Load configuration
 	./bin/confluent load jdbc_source_mysql_foobar_01 -d /tmp/kafka-connect-jdbc-source.json

	# Other useful commands are: 
	./bin/confluent status
	./bin/confluent unload
	./bin/confluent configure
	 



### Resources and sites I used to build this:

* [https://docs.confluent.io/current/quickstart/cos-quickstart.html#cos-quickstart](https://docs.confluent.io/current/quickstart/cos-quickstart.html#cos-quickstart)
* [https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/](https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/)
* [https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/](https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/)
* [https://debezium.io/docs/configuration/event-flattening/](https://debezium.io/docs/configuration/event-flattening/)
* [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)