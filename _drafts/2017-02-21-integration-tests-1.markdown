---
layout: post
title:  "Integration Tests, a practical approach"
date:   2017-02-21 23:52:00
categories: testing
tags:
 - PHP 
 - testing
 - phpunit
 - behat
 - httpful
---

When building and maintaining big web apps having a continuos integration is a must, and one of the key components is the automated testing.

> More reading about continuos integration:
> [https://www.martinfowler.com/articles/continuousIntegration.html]()


This is a practical guide about how to build a tool for automatically test our App.


But aren't unit tests meant to do that? -> NO unit test are great to help building robust software, and they will check portions of the software isolated but not the whole thing.

It's good to think about unit tests as a contract about how the tested unit should interact with the environment, and remember that we **mock** this environment. If your unit tests involve multiple parts of the system (Controllers, services, DB, logs), that means that you don't have unit test, just bad integration tests.


> More reading about Unit tests:
> [http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/]()

What we are trying to accomplish here is to:

* Focus manual testing on finding bugs, not doing repetitive tasks over and over.
* Make Developers detect regressions before depoying changes to production.


## Some thoughts about how the Automated-tests should be:

### 1. Testing tool should be independent of the target
Having separate apps ("Testing Tool" and "Target") will reduce the chances of having a bug that affects both Apps, even more, if we use a tool written in Ruby to Test a target app written in php it will even better. 

### 2. Testing tool should be able to start from a known state
The "Target" should have a known state and it can be restored to that state each time we start the testing session. 

### 3. Maintaining them should not be painful
Maintaining them should have a cost/benefit balance that encourages the team to have them.

### 4. The target should be as much as posible to production
Having bugs that only happens in production environments and not in the testing replicas is the nightmare, because of that, the replicas should be as close to the production instance: Same OS, hardware, deployment process, load and a long etc.


### 5. General Theory of Systems

The "Target" system could be considered as a "Black box", we will configure the initial state, and for each input we will try to find a consistent output.


## Practical approach I've found 

Automatic testing based on interactions through the HTTP API and other APIs that the application may rely on, Data Base APIs, logs, filesystem, email, third party tools.

Simple http client:

	composer require nategood/httpful 

BD driver (mongo):

	composer require mongodb/mongodb 

In order to maintain things organized, why not using a Behavior Driven Development tool to define the behavior of the API?

	composer require behat/behat
	
In order to increase the variability of our tests, why not including some entropy? instead or using the same input variables IE: for the addresses, why no use some random values generator like faker?

	composer require fzaninotto/faker


Once having all of that installed and configured, we can start writing behaviors, and then complete the tests interacting with the APIs and analyzing the result for anomalies.


## Even more Ideas

At this moment, we have built an API testing tool, but we can strength this approach including End-to-End test by using the mink extension of Behat connected to a selenium instance.

 
