---
layout: page
title: Disco Science IRL
date: 2023-03-26 00:00:00
description: A physical counterpart to the Factorio Disco Science mod
img: assets/img/disco-science.png
importance: 1
category: fun
---

**The source code and STL files for this project are available on [GitHub](https://github.com/noahcgreen/Disco-Science-IRL).**

[Disco Science](https://mods.factorio.com/mod/DiscoScience) is a great cosmetic Factorio
mod that should really be included in the base game. It doesn't impact gameplay in
any way; all it does is change the "glow" color of active labs to reflect the ingredients
that are being consumed by research. For example, the Speed Module technology consumes
Automation (red) and Logistic (green) science packs, so while you're researching that
tech, your labs will flash red and green.

Just for fun, and because LEDs are great, I decided to make a physical counterpart
to the in-game Disco Science labs to accompany me while I play. It came out really
nicely! The video below showcases it in use.

<div class="container">
    <div style="text-align: center">
        <iframe width="372" height="662" src="https://www.youtube.com/embed/7PQzcApEHyw" title="Disco Science IRL Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="caption">
    Disco Science IRL in action. When the researched tech is changed, the lab's colors change
    to match.
    </div>
</div>

## Reading In-Game Data With a Custom Mod

Disco Science colors labs to match the ingredients of the active research. I wanted to do the
same for my physical component, so I needed to query the game for those ingredients somehow
and deliver that information to my board. Fortunately, Factorio has incredible support for
modding, and practically anything going on in the game can be picked up by a mod. I followed
the approach of Disco Science to [read the ingredients of the tech being researched](https://github.com/danielbrauer/DiscoScience/blob/master/src/core/researchColor.lua).
Then, I sent the ingredients from the in-game Lua runtime to a file via the
[write_file](https://lua-api.factorio.com/latest/LuaGameScript.html#LuaGameScript.write_file)
API. It was surprisingly easy to do!

There was one caveat I encountered. Unfortunately, there doesn't seem to be a way for mods
to react to the game being quit/shut down, which would ideally be an event that my mod could
send outside of the runtime so the physical lab could turn itself off. To address this, I had
my mod write to an additional file once per second. If that file hasn't been touched in
more than one second, then the game has ended. Or so I thought! I then found out
that mods don't run when the game is paused or when the tech tree is being viewed, so my mod
would incorrectly make it appear as if the game had ended during those times. However, I'm
completely fine with that because research does not actually progress at those times, so it's
consistent for my lab to turn itself off then.

It does make sense that there is no "on game end/pause/etc." event for mods to
hook into, since a poorly written or malicious mod could delay shutdown and hurt the user's
experience.

## The Driver

The next step was to create a driver which would watch the files written by the mod and
forward the ingredients to the board as they come. Reading the files was straightforward;
most of the effort here went into managing the computer-board connection. I used the
third-party [PySerial](https://pyserial.readthedocs.io/en/latest/pyserial.html) Python
package for this task. I also used [py2exe](http://py2exe.org) to generate an executable
from the driver. I did this because I wanted the driver to run at all times without me
starting it manually from the command line. Once the executable was made, I copied a shortcut
to it into my Windows startup folder. Now it runs automatically when I log in.

## The Board

I started developing this project with an Arduino Nano, but I ended up switching to an
[Adafruit Flora](https://www.adafruit.com/product/659). It was the perfect choice since it
has USB-serial conversion and an addressable NeoPixel onboard, meaning I didn't need to solder
any additional components. That meant I could prototype very easily and will be able to
remove the board cleanly when I want to repurpose it. On top of that, the small circular
form factor meant it would fit easily into my 3D-printed enclosure.

The enclosure itself is based on this [lab keycap](https://www.thingiverse.com/thing:3240916)
I found on Thingiverse. I used Fusion 360 to hollow out the inside and add a bottom piece for
the Flora, then printed the pieces in transparent PETG. If I had the experience or
willingness, I might have painted the gray lines onto the lab to make it a bit more accurate,
but I didn't feel the need to this time.

<div class="container">
    <div class="row mt-3">
        <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/disco-science-irl/parts1.png" class="img-fluid rounded z-depth-1" %}
        </div>
        <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/disco-science-irl/parts2.png" class="img-fluid rounded z-depth-1" %}
        </div>
    </div>
    <div class="row mt-3">
        <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/disco-science-irl/parts3.png" class="img-fluid rounded z-depth-1" %}
        </div>
        <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/disco-science-irl/parts4.png" class="img-fluid rounded z-depth-1" %}
        </div>
    </div>
    <div class="caption">
        Parts, assembly, and activation!
    </div>
</div>
