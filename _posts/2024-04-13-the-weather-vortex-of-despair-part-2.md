---
title: "The Weather Vortex of Despair - Part 2"
date: 2025-04-13T20:00:00+01:00
categories:
  - blog
  - dev
tags:
  - Planets
  - Atmosphere
  - Environment
---

In my last post I detailed some highs and lows of trying to get working weather on a planet. High point - knowing I can calculate the temperature of a planet without an atmosphere. Low point - realising I was getting hopelessly lost trying to figure out how to do the same trick with an atmosphere.

Now I know I must be in trouble because I turned to AI for help. As I often find at the moment (and I don't know how anyone uses AI to do anything useful in coding that actually saves time), it sort of gaslit me and gave me something that didn't work but looked pretty close to something that *might* work, but the more I tried to fix things (either by manually editing or telling the AI what it had done wrong), the more the code started to look like complete junk. I thought it had some good ideas in there and I spent many, many hours iterating over those ideas thinking I'd get to somewhere good - in the end I had a mad planet with extremely violent winds, massive swings in temperature and just no survivability rating.. and it was still zero degrees Kelvin on the dark side, just with these really high winds that made no sense. I went back to the original code it had given me and realised, annoyingly, it was rubbish to begin with and I hadn't noticed.

For me, it's sometimes when I kind of give up and take a step back that I then find the solution I was looking for, and I have to say I did start to give up and try to work on other areas of Itinerant instead - but I couldn't get out of my head that it surely cannot be impossible to figure out some decent weather for my planet. I had to admit something I knew at the start of this - I do not have a degree or PhD in climatology, fluid dynamics, thermal dynamics, or any of that stuff that would actually help me figure this out, I don't have supercomputers to simulate real weather patterns... and I don't yet have access to an AI model that can do it for me. Maybe one day I will.

But I got to thinking that really there must be some kind of solution out there that could help me, and my research of the past few weeks at least meant I'd learnt a few key words I could throw into search engines and maybe guide me to something useful. I forget which words really helped but the next time I'm at a party with a bunch of scientists (which is proabbly never going to happen) I can now nod knowingly if someone says something like 'Boltzmann Constant' or 'Lapse rate', 'Relative vorticity', specific humidity vs. relative humidity... I am going to be so much fun next time I meet some real live people at a party and they ask me what I've been working on lately.

So I thought what I needed was an open source weather simulator that could help me out... and then I found a piece of software called SpeedyWeather.

Now this managed to unfurl a whole *other* can of worms because it's written in Julia, which is a language I don't think I'd heard of before, so suddenly I traded my learning about weather with learning about Julia, and wondering if I could get it to interact with C# in a way that made sense (I can feel the weather info getting pushed out of my brain and getting replaced with Julia already)... but what I did manage to do with SpeedyWeather is feed it my land terrain data, a land-sea mask so I could tell it where water was on the planet, some parameters about the size and rotation of the planet.. and, lo and behold, it started simulating something for me that looked like actual weather!

I'm genuinely impressed by what it can do, especially considering it's available under an MIT license and the people involved in the project seem very, very helpful and super engaged on GitHub. Often you find these kinds of open source projects and then realise the last commit was several years ago, the issues tab has someone asking 'has this been abandoned', the one thing you really need to work doesn't, and the code is so complex that you don't stand a chance of adding what you need. Not so here. It just works, and it works well!

So here we have it. This is a bit of a milestone day because I set up a service on my dev machine that simulates the weather on my planet and consistently updates it in 30 second increments. I know wind speeds, surface pressure, and temperatures as different layers above the planet's surface and I'm recalculating it in close to real-time (with a fair bit of interpolation going on at times, to be fair - I set up SpeedyWeather to give me conditions every hour and then interpolate the conditions from one hour to the next).

So I now have a background service running that I can add to going forwards and keep my planet evolving in real-time. It's not just that this means I have a route to getting working weather on the planet, it means I have a way of interacting and modifying terrain and the stuff the player is building as the game progresses. I want stuff like your base to get covered in dust when it's windy and dry, or leaks to appear if it's raining. This is now possible even when the player is away from the game and it opens up a lot of doors in terms of making the universe I want to create more dynamic and 'real'.

{% include figure popup=true image_path="/assets/images/blog/proxima-d-surface-temp.gif" alt="Surface temperature changing over a 24-hour period." caption="Surface temperature changing over a 24-hour period." %}

{% include figure popup=true image_path="/assets/images/blog/proxima-d-rel-vorticity.gif" alt="Relative vorticity changing over a 24-hour period." caption="Relative vorticity changing over a 24-hour period." %}
