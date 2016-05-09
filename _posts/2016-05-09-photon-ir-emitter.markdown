---
layout: post
title:  "IOT: Photon based IR-emitter"
date:   2016-05-09 12:36:00
categories: IOT 
tags:
 - IOT 
 - Infrared
 - Remote
 - Photon
---


### Idea: Connect any device that uses a remote control to internet.

Why I would do that?

* To start cooling my apartment before I arrive. ([as this example](http://cmaher.github.io/posts/developing-with-the-particle-photon/))
* To control the webcam over a hangouts call.
* To turn on and off the lights when I'm not in home.
* Because I can :)

>First things first, what is a photon?: is like an Arduino but wifi-connected.

So, as it pretty close to the Arduino, it is simple to adapt sketches from one platform to the other. Actually there is an Arduino project to control stuff using IR ([Arduino-IRremote](https://github.com/z3t0/Arduino-IRremote)) and a port of it to the Particle-core platform ([Spark-Core-IRremote](https://github.com/qwertzguy/Spark-Core-IRremote)) 

In this project I set up both:

* The Arduino is for recording the codes 

![arduino schema](/files/iot-ir/receiver_bb.png)

* And the photon is the emitter

![photon schema](/files/iot-ir/emitter_bb.png)

![photon photo](/files/iot-ir/photon-led.gif)

In the Arduino I uploaded the example IRrecvDumpV2 to record the codes that later I pass to the Photon.

After some trial and error I mapped all the buttons of this camera:

![Camera c6000](/files/iot-ir/hangouts-implementation-camera.png)

Resources:

* [http://cmaher.github.io/posts/developing-with-the-particle-photon/](http://cmaher.github.io/posts/developing-with-the-particle-photon/)
* [https://github.com/z3t0/Arduino-IRremote](https://github.com/z3t0/Arduino-IRremote)
* [https://github.com/qwertzguy/Spark-Core-IRremote](https://github.com/qwertzguy/Spark-Core-IRremote)



 
