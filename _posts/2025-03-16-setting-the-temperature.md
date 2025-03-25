---
title: "Setting the Temperature"
date: 2025-03-16T15:34:30-04:00
categories:
  - blog
tags:
  - Planets
  - Atmosphere
  - Environment
---

One of the things I'm aiming to achieve with Itinerant is a fairly decent set of survival mechanics. So far, I've mainly been concentrating on making sure the environment I'm creating is capable of generating realistic-looking terrain and subterranean geology, but this weekend I turned my attention to something I've been meaning to add - knowing what the surrounding temperature is when on a planet.

I was initially torn about the way to approach this - with my Earth planet, it's very tempting to just take some current actual weather observations and plug them into the game and then just jumping ahead to dealing with how that data effects the player and surrounding area - however I felt a bit like that's putting the cart before the horse. This entire project has always been about getting the hard work done first to reap the rewards later rather than finding shortcuts that make things look good early on but just give me a hard-coded mess with unresolved stuff that needs sorting out later.

That said, I'm also aware that realistic weather simulations involve supercomputers, people with PHDs and decades of research, which is somewhat out of my reach, so I really can't go full-on *'I'm going to simulate everything using physics! In detail!'*

I need to find a good path through the middle which gives me a decent base to build on and can be compared to real Earth data to see if I'm getting anywhere close to reality. As a side-note, I also want to abstract this all away so different planets can use different environment models where needed. When the time is right and we want to look at a fully-traversable real-time clone of planet Earth, it will indeed rain when it's raining outside, and then I'll need the real-time weather data I'm currently trying hard to ignore.

Obviously there's a lot to figure out, but I decided a good place to start would be to see if I could find a reasonable 'average' temperature for any planet based on some known factors (like size/brightness of it's star, distance of the planet to the star and the size of the planet) and then just see what direction that took me in.

There's some interesting articles about planetary temperatures on the web, and it became apparent that there are some simple calculations you can do to make rough estimates that don't seem too far removed from reality.

The first thing I came across was this Stack Exchange question which had a lot of symbols I didn't initially understand:

https://astronomy.stackexchange.com/questions/10113/how-to-calculate-the-expected-surface-temperature-of-a-planet

The answer to the question mentions 'radiative equilibrium temperature' (aka Planetary Equilibrium Temperature), which is the average temperature a planet would be at if you consider it a black body radiator with no greenhouse effect caused by the atmosphere. So this is kind of my introduction to global weather.

This formula in particular is a good start:

![image](https://github.com/user-attachments/assets/aff2c930-e189-4387-ba5f-bff79ddf8499)

It mentions using the luminosity in Watts (L), the distance from the star in meters (d), the Stefan-Boltzmann constant (that's the Greek symbol sigma, you can read more about what it is here), and the albedo of the planet (a).

Therefore my two initial thoughts were, how do I get the luminosity of the star, and what is the albedo (other 3D programming-speak for the color of a thing).

#### Luminosity 

It turns out this can be calculated by knowing the temperature of the star, and there is a direct relationship between the mass of the star and it's temperature... which all seems pretty obvious now I think of it - basically the bigger the star, the brighter it is. This is great for my procedurally-generated universe, because I can just randomly determine the size of the star and I'll automatically know how bright it is and the color of the light it radiates.

Looking to this page:

https://www.astronomy.ohio-state.edu/thompson.1847/1144/Lecture9.html#:~:text=L%20%3D%20F%20x%20Area%20%3D%204,power%20and%20its%20Radius%20squared.%22

I found a way to determine the luminosity:

![image](https://github.com/user-attachments/assets/d27a6b38-c2f4-4155-b8cc-8209453f9773)

So, all we need to know to get the luminosity of the star is its radius and its temperature. Everything else is just Pi and the Stefan Boltzmann constant, which we already learned about earlier.

#### Albedo

Albedo as I know it is just the raw colour of something without any shading or lighting altering it. We use it a lot in 3D, especially in regards to texture maps. However, the albedo in this case is a bit different, and somewhat simplified.

We can view albedo as being the amount of energy reflected back into space, expressed as a ratio between 0 and 1. When some of a star's light hits a planet, a certain amount of that light will bounce back into space and won't get converted into heat - so when we consider how much a planet is heated up by a star, in very simple terms we're multiplying the sun's energy that reaches the planet by one minus the albedo, which gives us the remaining amount of energy that is absorbed into the surface.

Now of course a planet is made up of many different materials, and they all have their own albedo.

Wikipedia has a good list of typical albedos:

Down the line I think all my materials in my web interface are going to need an albedo level. I could then run some sort of calculation based on the composition of the planet to work out it's resultant albedo.

I feel like filling out a database with albedos (or working out a way to calculate what the albedo for each material might be, which is almost certainly going to be a massive data entry task however I do it) is something to be done down-the-line so for now I think it's reasonable to use some known albedos to help me out. I know, this is a bit of a shortcut but it's certainly something that can be revisited later without massively altering my code.

The Earth has an albedo of 0.612 so you know what, I've only got the weekend to do this, so that's what I'm going to use for planets.

#### Putting it togther

Handily this all starts to make some sense to me. Going back to that previous equation, we know the following:

- The luminosity of the star
- The size of the planet
- How far the planet is from the star
- How much of the star's energy gets reflected back into space by the planet

Given those factors, we also know that only one half of the planet is being lit by the star at any one time (so we can divide the energy from the sun by two), and then we also know that the sun hitting the planet doesn't hit equally across the entire lit surface, because the planet is spherical (so we can divide the energy by 2 again).

This is how we end up with the original formula. As shown on Stack Exchange, plugging data for the Earth into it gives this:

![image](https://github.com/user-attachments/assets/a04672a2-dfea-41f6-8a0d-892cf169af33)

This works out to about 256K, or -17°C.

Now of course, we have a slight problem because it's not that cold on planet Earth. So far, we haven't considered the atmosphere at all. Still, it's a good starting point - there are likely to be many planets in Itinerant where there is no atmosphere, and we're getting close to being able to figure out whether you or your spaceship is going melt when you try to touch down. Now it'd just be nice to try to add some sort of estimate for the atmosphere.

#### Emissivity

There is a really, really simple way to make our formula retain more heat and get it closer to the temperature on Earth - Emissivity.

Emissivity is a ratio we can use to help us decide how much heat gets retained on the planet. I can't confess to understand it fully, but I think it's fair to view it as a mix of a number of different things - for example, CO2 in the atmosphere contributes to emissivity because it creates a greenhouse effect that traps heat within the planet. Oceans also contribute to emissivity because they soak up heat and only release it slowly. Pretty much anything on the planet's surface and atmosphere can contribute to it, and when it's all put together you can figure out an average emissivity value for a planet.

In the future I can see that what I can do is add emissivity values to any material in the database, similar to albedo and then we'd be able to calculate emissivity values for an entire planetary body reasonably accurately (bearing in mind we also need to consider the atmosphere as well). For now, I think it's OK to use a known emissivity value for the Earth and then we can deal with the how we calculate emissivity as a future improvement.

This gives us our final formula:

![image](https://github.com/user-attachments/assets/a985b017-6e64-42d3-b37b-148086086199)

Where 'e' is emissivity.

After all that, what we have here is just a global average temperature for a planet. It may be a reasonable estimate, but how do we know the temperature at a specific point on the planet and for that matter, what about a specific time of day/night?

I'll be covering this in the next post, and hopefully plugging it into Itinerant to see it in action.
