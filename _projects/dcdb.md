---
layout: page
title: DCDB
date: 2023-03-03 00:00:00
description: Simulation Engine and GUI Application for the DC Deck-Building Game
img: assets/img/card-back.jpeg
importance: 1
category: fun
---

[Source code on GitHub](https://github.com/noahcgreen/dcdb)

**dcdb** is a neat project that I worked on for fun in college. It's a simulation engine
for the [DC Deck-Building Game by Cryptozoic Inc.](https://cryptozoic.com/products/dccdbg)
My goal in this project was to capture this game's complex logic into a library which could
then be used to build an app or a headless server for multiplayer.

The main challenge I faced in writing this library was presented by the generality of
DCDBG's rules. The basic framework of the game is simple; there's a turn order, players
can play cards, buy cards, etc. However, the game becomes massively more complicated when
cards' abilities are considered. Each card (and there are many) has an ability which affects
gameplay, often in ways not mentioned in the rulebook. I wanted to make sure that my library
could support all of the official cards and potentially even custom cards as well.

With this in mind, I implemented this library with the following structure:

* The core game logic is represented by *[routines](https://github.com/noahcgreen/dcdb/blob/main/dcdb/routines.py)*, small pieces of logic which pause
themselves when user input is required. Routines can also yield control to other routines.
In a traditional game engine, routines might be implemented as state machines on a stack,
where the game continuously resumes the topmost routine with the latest piece of user inputs.
I chose to implement routines using Python generators, which offer a very convienent way of
writing code which can be paused and resumed with inputs. Routines yield to each other by
literally "yielding" to sub-generators, using recursion as the traditional stack.
* All card abilities are implemented through a [Lua scripting engine](https://github.com/noahcgreen/dcdb/tree/main/dcdb/scripting) rather than in the game's
logic. This design allowed the engine to be minimalistic and complete without requiring that
all cards and corner cases be considered from the onset.

I also wrote a simple [GUI application](https://github.com/noahcgreen/dcdb/tree/main/dcgui) for the library in Kivy, using a pub-sub [observer pattern](https://github.com/noahcgreen/dcdb/blob/main/dcdb/observe.py) to watch game events and update the GUI as they occurred.

Someday I hope to rewrite this project in C++ since Lua is easier to work with there and it'll integrate better with various GUI frontends.
