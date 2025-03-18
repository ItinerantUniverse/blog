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

Handily this all starts to make some sense to me. As mentioned earlier, we're saying the luminosity of the star is in Watts, which I know as a unit of power. So, going back to that previous equation, we can see what we're really doing is figuring out how much power hits the planet at any given moment, bouncing a bunch back thanks to albedo and converting the rest into heat.

This all led me onto the next article:

https://web.archive.org/web/20210605120431/https://scied.ucar.edu/earth-system/planetary-energy-balance-temperature-calculate

Now here I find things are getting really interesting, because the article establishes a fairly basic rule - the planet absorbs a certain amount
of energy, and reflects a certain amount back into space, and it hints at the fact you can figure this out not only across the entire surface of
the planet, but for any surface area.

This got me thinking that down-the-line, I can probably come up with a base value for the tmeperature, but then when the player is actually
standing on the surface, we can calculate the amount of energy hitting the player in realtime. We could use this to figure out when they're
getting too hot or too cold, or maybe even getting sunburnt, and we could make this a cumulative effect that just constantly happens to the player
and their stuff while they're on the surface. To that end, their spaceship while they're in space could be subject to the same effect. We don't
necessarily just need to hard-code a distance to the sun where the ship burns up, we can instead rely on physics to burn the ship up for us :D

Back to this initial stage, I felt it wasn't good enough to just get a value for an average temperature across the entire planet, because really
the temperature should vary based on where you're standing, and of course it'll also vary if it's night or day. This led me to another post:

https://worldbuilding.stackexchange.com/questions/185703/how-to-determine-planetary-extremes-of-temperature-from-average-global-temperatu

This one I found really great, because it includes a way to figure out where the sun will be hitting based on axial tilt of the planet, and
since my planets already have axial tilts on them, I thuoght it'd be amazing to try to combine everything I've learnt together.

From that last link, I ended up writing three functions to help me determine how much energy falls on a particular part of the planet on a
particulr day:

        public double GetEffectiveLatitude(double actualLatitude, double yearRatio)
        {
            return actualLatitude + _planet.AxialTilt.Z * Math.Sin(yearRatio * 2 * Math.PI);
        }

        public double GetSolarPower(double actualLatitude, double yearRatio)
        {
            return Math.Abs(Math.Cos(GetEffectiveLatitude(actualLatitude, yearRatio)));
        }

        public double GetSolarPower(WGS84 pos, DateTime atTime)
        {
            TimeSpan t = atTime - DateTimeOffset.UnixEpoch;

            return GetSolarPower(pos.LatitudeRadians(), _planet.OrbitalPeriod / t.TotalDays);
        }

I think I need to find a better name for it, but basically 'GetSolarPower' returns a value between 1 and 0, where 1 getting the full
force of the sun's rays, and 0 is obviously getting nothing at all, because it's facing directly upwards relative to the star.

[An improvement on this would be to also include the time of day, so the solar power falls away as the planet rotates - this is something
I'll definitely add.]

To test this is working, I overlaid my planet with color values to show how the power changes across the surface of the planet. In the
image below, red represents the areas getting the most energy, and green the least:

--

I was amazed to see that it actually came out looking roughly right on a first try - usually i get these things wildly wrong first time
and then have to spend hours trying to figure out what the issue is before finding I copied a formula down wrong somewhere!

I then have this functino to determine a baseline temperature:

        public double GetEquilibriumTemperature()
        {
            return Math.Pow(((1 - _albedo) * StarLuminosity()) / (16 * Math.PI * _emissivity * GenericPlanet.STEFAN_BOLTZMANN * Math.Pow(_planet.OrbitalDistance, 2)), 1 / 4.0);
        }


My plan is to use these formulas alongside something that makes a 'best guess' based on the type of atmosphere, presence of water
on the surface and height of the terrain to generate some sort of heatmap that can be precalculated for every procedural planet.

In this way, I think I'll be able to work out how temperature fluctuates between day and night, but also make smaller fluctuations based on
cloud cover, weather conditions, etc.
