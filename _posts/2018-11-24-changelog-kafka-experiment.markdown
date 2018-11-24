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

#### 1) get a legacy app based on MYSQL database

The application should follow some rules:

* Have a MYSQL database
* Tables should have an `last_updated_at` timestamp column or an incremental revision id that automatically updates on every row save
* MYSQL binlog should be enabled

if you don't have any app like that, this is as simple drupal example to load with `docker-compose`

    # Use root/example as user/password credentials
    version: '3.1'
    
    services:
    
      drupal:
        image: drupal:8-apache
        ports:
          - 8080:80
        volumes:
          - /var/www/html/modules
          - /var/www/html/profiles
          - /var/www/html/themes
          - /var/www/html/sites
        restart: always
    
      db:
        image: mysql
        command: --default-authentication-plugin=mysql_native_password
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_ALLOW_EMPTY_PASSWORD: "true"
          MYSQL_DATABASE: test
    
    
      adminer:
        image: adminer
        restart: always
        ports:
          - 8080:8080
      
      

### Resources and sites I used to build this:

* [https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/](https://rmoff.net/2018/03/24/streaming-data-from-mysql-into-kafka-with-kafka-connect-and-debezium/)
* [https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/](https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/)
* [https://debezium.io/docs/configuration/event-flattening/](https://debezium.io/docs/configuration/event-flattening/)