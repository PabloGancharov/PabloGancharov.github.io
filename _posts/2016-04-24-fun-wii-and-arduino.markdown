---
layout: post
title:  "Adapting PlayStation 2 controller to Wii using Arduino"
date:   2016-04-24 15:40:56
categories: Arduino
tags:
 - Wii 
 - Classic Controller
 - Play Station 2 
 - PS2
---

### The Problem

Some days ago I bought a pair of Wii Classic controllers to play Mario Kart 64 on the Nintendo Virtual Console, but unfortunately I spent my money on a Chinese imitation, instead of buying the originals. 

> The main reason is that I don't wanted to wait 2 months for a package to arrive from US to Uruguay 
> 
> The secondary reason was the price, the pair of Chinese controllers cost then time less than the originals.

In fact, the controllers quality was awful, and one of it stopped to work after some use.

### The Solution

After google-ing a little I found 2 repos: 

To communicate with the Wii-mote:
[https://github.com/denizkoekden/arduino-wiimote](https://github.com/denizkoekden/arduino-wiimote) that was created 4 years ago, but updating it wasn't complicated.

and for the PlayStation Controller I found the excellent work of Bill Porter: [http://www.billporter.info/2010/06/05/playstation-2-controller-arduino-library-v1-0/ ](http://www.billporter.info/2010/06/05/playstation-2-controller-arduino-library-v1-0/ ) 


[https://github.com/madsci1016/Arduino-PS2X](https://github.com/madsci1016/Arduino-PS2X) 


**The goal was mix both project to create an adapter from PS2 to Wii (╯°□°）╯︵ ┻━┻**

Basically what I did was mixing these two projects and matching all the keys from the PS2 to impersonate a Classic Wii controller.

![Arduino connections](/files/ps2-wii/arduino-pins_bb.png)

### Results:

Development Layout

![Dev Layout](/files/ps2-wii/2016-04-17 21.56.05.jpg)

Playing Mario Kart 64

![Playing Mario Kart 64](/files/ps2-wii/mario-kart-demo.gif)

### Future improvements:
* Remove the 5V power supply and make all run with the Wii-mote batteries.
* Use a female connector to allow switching the PlayStation Controller.
* Put all together in a pretty case like this:

![Altoids Arduino Case](/files/ps2-wii/altoids.png)

### Assets:

* Arduino final source code (working on Arduino 1.6.8): [wii-psx.zip](/files/ps2-wii/wii-psx.zip) 
* Fritzing file: [arduino-pins.fzz](/files/ps2-wii/arduino-pins.fzz) 

