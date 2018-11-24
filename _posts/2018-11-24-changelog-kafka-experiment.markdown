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

Mission: 
1) Emmit events everytime data changes in an existing application, without touching the original codebase.
2) Use those events to create a live changelog of the app

High level milestones:

1) get a legacy app based on MYSQL database

2) Setup Kafka, Kafka connect and debezium

3) Configure Kafka Connect to detect changes on specific tables 

Extra goals: 

1) Configure Kafka Connect to detect changes on aggregated data (Joined tables)

2) Create a custom transformation to modify the generated output




### Virtual retrospective tools:


### Technical Details:
<!-- 	
	Hardware:
		Thermal printer (https://www.adafruit.com/product/597)
		Particle Photon (https://store.particle.io/)

	UI to send postits:
		Simple HTML form -->

### Code to flash the photon:
<!-- 
[https://gist.github.com/PabloGancharov/73c4729798d338b4507512e76f91d922.js](https://gist.github.com/PabloGancharov/73c4729798d338b4507512e76f91d922.js)
 -->

### UI:
<!-- 
> Note: current UI implementation is just a proof of concept, never share your device id or security token publicly like this:

[https://gist.github.com/PabloGancharov/5c919e2671119fe5fdba400eaf7fb135](https://gist.github.com/PabloGancharov/5c919e2671119fe5fdba400eaf7fb135)
 -->
<!-- ![UI Example](/files/Screen Shot 2018-08-04 at 12.54.58.png) -->

### Results:
<!-- 
<video width="480" height="320" controls="controls">
  <source src="/files/2018-04-11 11.41.48.mp4" type="video/mp4">
</video>
 -->
