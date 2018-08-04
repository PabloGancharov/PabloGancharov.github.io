---
layout: post
title:  "Improving 'agile retrospectives' experience on remote teams by using a IoT Postit printer"
date:   2018-07-08 00:00:00
categories: Agile iot
tags:
 - Agile
 - IOT
 - post-it
 - printer
 - spark photon
---

### Problem: 
Agile retrospectives are key element of agile teams to enable self analysis and process improvement.
But today, it is really common to have people working from remote locations, so how can we leverage everybody’s participation when there are some remote folks in the team?

### Virtual retrospective tools:
Web applications are a common solution that simulate all the elements required of that meeting: the board, the posits and the allowed interactions between the human beings like grouping items and voting.

When we prepare a retrospective, the main goal is to generate valuable feedback from all the participants, we can do this by using, ice breaker games, encouraging everybody to use the 'critital hat', and providing a safe place to allow everybody to express themselves without being judged. The mood is to discover things we would like to improve and think how to achieve that as a team.

It’s difficult to generate that mindset, so different from daily work, just opening another page tab in the browser. Even worst if we force everyone to communicate on the computer just as if all team members were working remotely.

I used to work on a remote team, I mean one with team members dispersed around the world, fortunately in the same timezone. We were half based in Uruguay (all the developers) and half based in United States (the PO and the designer).

At the beginning we used 'mura.ly' to organize our retrospective board,
but this idea came up when we tried to do a simple and short retrospective without the computers or cellphones around, and just using google meetings, webcams, real post-its and a real whiteboard. At that time we discovered how much easy it was to think about our past sprint, how we felt and propose ideas about how to improve. The only drawback was how to integrate the notes from the 2 remote peers in the whiteboard

At that time the solution was simple: remote people sent their feedback by chat to the scrum master and he put the items on the wall while the author explained the meaning. By having such a different and simple experience about writing down our thoughts on paper and posting them on the wall, we came up with a simple improvement, how to automate that process?, by using a remote post-its printer of course 🙂


### Technical Details:
	
	Hardware:
		Thermal printer (https://www.adafruit.com/product/597)
		Particle Photon (https://store.particle.io/)

	UI to send postits:
		Simple HTML form

### Code to flash the photon:

[https://gist.github.com/PabloGancharov/73c4729798d338b4507512e76f91d922.js](https://gist.github.com/PabloGancharov/73c4729798d338b4507512e76f91d922.js)


### UI:

> Note: current UI implementation is just a proof of concept, never share your device id or security token publicly like this:

[https://gist.github.com/PabloGancharov/5c919e2671119fe5fdba400eaf7fb135](https://gist.github.com/PabloGancharov/5c919e2671119fe5fdba400eaf7fb135)

![UI Example](/files/Screen Shot 2018-08-04 at 12.54.58.png)
