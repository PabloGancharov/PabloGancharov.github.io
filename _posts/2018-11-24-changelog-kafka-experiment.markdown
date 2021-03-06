---
layout: post
title:  "Adding change-log to a legacy mysql based application using Kafka and Debezium"
date:   2018-11-24 00:00:00
categories: Kafka Changelog Mysql Debezium
tags:
 - Kafka
 - Changelog
 - Mysql
---


Adding change-log to a legacy APP using Kafka and Debezium (without modifying the codebase)

### Mission: 

1. Emmit events everytime data changes in an existing application, without touching the original codebase.
2. Use those events to create a live changelog of the app

### High level milestones:

1. Get a legacy app running (MYSQL based)
2. Setup Kafka and Debezium


### Let´s start


#### STEP #1 - Get a legacy app based on MYSQL database

The application should follow some rules:

* Have a MYSQL database
* MYSQL binlog should be enabled

If you don't have any app like that, this is as simple drupal example to load with `docker-compose`

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

#### STEP #2 - Setup Kafka and debezium

For our purposes we will use the __Confluent Open Source__ Kafka quick start bundle  [https://www.confluent.io/download/](https://www.confluent.io/download/)

1. Download and unzip the confluent package (in my case `confluent-oss-5.0.1-2.11.zip`)
2. Add Debezium plugin (this will be usefull to also apply changes to the streamed data). Download the latest from [https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/](https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/) (in my case `debezium-connector-mysql-0.9.0.Alpha2-plugin.tar.gz`) unzip it and move the extracted folder to `share/java/`



Start Kafka: `./bin/confluent start`

![Confluent start](/files/kafka-changelog/confluent_start.png)

Also you can access [http://localhost:8083/](http://localhost:8083/) to check the rest API is working.

__Testing producer/consumer w/ simple messages__

let's use 2 teminal sessions to teest kafka is working fine:

In one terminal start the Kafka avro console consumer: `./bin/kafka-console-consumer  --bootstrap-server localhost:9092 --topic test --from-beginning`


Then in another terminal, let's start the producer: `./bin/kafka-console-producer --broker-list localhost:9092 --topic test` 

At this point you can type anything and every time you hit enter the message will be sent to the `test` topic and will appear in the `console consumer`

![Testing Kafka producer and consumer are working](/files/kafka-changelog/test_kafka.gif)

Stop the  producer hit Ctrl + C.

__Configure Debezium to listen database changes__


First create a new configuration file `/tmp/debezium-source.json`
with the conection details and the whitelist of tables to listen.


	
	{
	  "name": "debezium-source",
	  "config": {
	    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
	    "tasks.max": "1",
	    "database.hostname": "localhost",
	    "database.port": "23306",
	    "database.user": "root",
	    "database.password": "root",
	    "database.server.id": "184054",
	    "database.dbname" : "drupal_db",
	    "database.server.name": "changelogExample",
	    "database.history.kafka.bootstrap.servers": "localhost:9092",
	    "database.history.kafka.topic": "dbhistory.drupal_db",
	    "snapshot.mode" : "when_needed"
	  }
	}
	

and load that configuration into Connect:

	./bin/confluent load debezium-source -d /tmp/debezium-source.json

Other useful commands are: 

	# get the status
	./bin/confluent status debezium-source
	
	# unload config
	./bin/confluent unload debezium-source

	# relaod config
	./bin/confluent config debezium-source -d /tmp/debezium-source.json
	
	
Let's see the topics list:
	
	./bin/kafka-topics --zookeeper localhost:2181 --list
	
<details>
  <summary>Click to see results</summary>
  
  
	__confluent.support.metrics
	__consumer_offsets
	_confluent-ksql-default__command_topic
	_schemas
	changelogExample
	changelogExample.drupal_db.block_content
	changelogExample.drupal_db.block_content__field_banner_image
	changelogExample.drupal_db.block_content__field_content_link
	changelogExample.drupal_db.block_content__field_copyright
	changelogExample.drupal_db.block_content__field_disclaimer
	changelogExample.drupal_db.block_content__field_promo_image
	changelogExample.drupal_db.block_content__field_summary
	changelogExample.drupal_db.block_content__field_title
	changelogExample.drupal_db.block_content_field_data
	changelogExample.drupal_db.block_content_field_revision
	changelogExample.drupal_db.block_content_revision
	changelogExample.drupal_db.block_content_revision__field_banner_image
	changelogExample.drupal_db.block_content_revision__field_content_link
	changelogExample.drupal_db.block_content_revision__field_copyright
	changelogExample.drupal_db.block_content_revision__field_disclaimer
	changelogExample.drupal_db.block_content_revision__field_promo_image
	changelogExample.drupal_db.block_content_revision__field_summary
	changelogExample.drupal_db.block_content_revision__field_title
	changelogExample.drupal_db.cache_bootstrap
	changelogExample.drupal_db.cache_config
	changelogExample.drupal_db.cache_container
	changelogExample.drupal_db.cache_data
	changelogExample.drupal_db.cache_default
	changelogExample.drupal_db.cache_discovery
	changelogExample.drupal_db.cache_entity
	changelogExample.drupal_db.cache_menu
	changelogExample.drupal_db.cache_render
	changelogExample.drupal_db.cachetags
	changelogExample.drupal_db.config
	changelogExample.drupal_db.content_moderation_state
	changelogExample.drupal_db.content_moderation_state_field_data
	changelogExample.drupal_db.content_moderation_state_field_revision
	changelogExample.drupal_db.content_moderation_state_revision
	changelogExample.drupal_db.file_managed
	changelogExample.drupal_db.file_usage
	changelogExample.drupal_db.history
	changelogExample.drupal_db.key_value
	changelogExample.drupal_db.key_value_expire
	changelogExample.drupal_db.menu_tree
	changelogExample.drupal_db.node
	changelogExample.drupal_db.node__body
	changelogExample.drupal_db.node__field_cooking_time
	changelogExample.drupal_db.node__field_difficulty
	changelogExample.drupal_db.node__field_image
	changelogExample.drupal_db.node__field_ingredients
	changelogExample.drupal_db.node__field_number_of_servings
	changelogExample.drupal_db.node__field_preparation_time
	changelogExample.drupal_db.node__field_recipe_category
	changelogExample.drupal_db.node__field_recipe_instruction
	changelogExample.drupal_db.node__field_summary
	changelogExample.drupal_db.node__field_tags
	changelogExample.drupal_db.node_access
	changelogExample.drupal_db.node_field_data
	changelogExample.drupal_db.node_field_revision
	changelogExample.drupal_db.node_revision
	changelogExample.drupal_db.node_revision__body
	changelogExample.drupal_db.node_revision__field_cooking_time
	changelogExample.drupal_db.node_revision__field_difficulty
	changelogExample.drupal_db.node_revision__field_image
	changelogExample.drupal_db.node_revision__field_ingredients
	changelogExample.drupal_db.node_revision__field_number_of_servings
	changelogExample.drupal_db.node_revision__field_preparation_time
	changelogExample.drupal_db.node_revision__field_recipe_category
	changelogExample.drupal_db.node_revision__field_recipe_instruction
	changelogExample.drupal_db.node_revision__field_summary
	changelogExample.drupal_db.node_revision__field_tags
	changelogExample.drupal_db.router
	changelogExample.drupal_db.search_dataset
	changelogExample.drupal_db.search_index
	changelogExample.drupal_db.search_total
	changelogExample.drupal_db.semaphore
	changelogExample.drupal_db.sequences
	changelogExample.drupal_db.sessions
	changelogExample.drupal_db.shortcut
	changelogExample.drupal_db.shortcut_field_data
	changelogExample.drupal_db.taxonomy_index
	changelogExample.drupal_db.taxonomy_term__parent
	changelogExample.drupal_db.taxonomy_term_data
	changelogExample.drupal_db.taxonomy_term_field_data
	changelogExample.drupal_db.url_alias
	changelogExample.drupal_db.user__roles
	changelogExample.drupal_db.users
	changelogExample.drupal_db.users_field_data
	changelogExample.drupal_db.watchdog
	connect-configs
	connect-offsets
	connect-statuses
	dbhistory.drupal_db
</details>

And now check the changelog of one of the tables:

	# Please note I'm using jq tool to format json output :
	./bin/kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic changelogExample.drupal_db.node__body --from-beginning | jq "."	
	 
[![Testing Kafka connect](/files/kafka-changelog/debezium.gif)](/files/kafka-changelog/debezium.gif){:target="_blank"}



### Final Notes

Despite this example is a proof of concept about relational databases and changelogs, I want to highlight the power and flexibility of Kafka and Debezium. 

Other uses of events generated by Debezium: 

(extracted from [https://debezium.io/docs/faq/#what_are_some_uses_of_debezium](https://debezium.io/docs/faq/#what_are_some_uses_of_debezium))

* Update search indexes with the data.
* Update a derived data store with the same information or with information computed from the changing data, such as with Command Query Responsibility Separation __(CQRS)__.
* Send a push notification to one or more mobile devices.
* They might aggregate the changes and produce a stream of patches for entities.

### Resources and sites I used to build this:

* [https://docs.confluent.io/current/quickstart/cos-quickstart.html#cos-quickstart](https://docs.confluent.io/current/quickstart/cos-quickstart.html#cos-quickstart)
* [https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/](https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/)
* [https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/](https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/)
* [https://debezium.io/docs/configuration/event-flattening/](https://debezium.io/docs/configuration/event-flattening/)
* [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)
* [https://iamninad.com/how-debezium-kafka-stream-can-help-you-write-cdc/](https://iamninad.com/how-debezium-kafka-stream-can-help-you-write-cdc/)