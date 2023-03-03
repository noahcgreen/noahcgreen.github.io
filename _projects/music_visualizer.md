---
layout: page
title: Music Visualizer
date: 2022-05-04 00:00:00
description: An OpenGL application for visualizing MIDI input in real time
img: assets/img/music-vis-preview.png
importance: 1
category: school
---

The following is an application I developed for my Computer Graphics final project in
the Sprint 2022 semester. It's a small program that reads MIDI input over USB from a
digital piano and shows the notes played in real-time, together with dynamics, note length,
and a waveform display. It's written from scratch in pure C++ using only raw OpenGL for
graphics (plus GLFW for windowing).

This project helped reinforce my understanding of low-level computer graphics, including
shaders and buffering. It was also neat to implement the waveform view, which calculates
an additive wave from [musical note frequencies](https://en.wikipedia.org/wiki/Musical_note#Note_frequency_(in_hertz)).

The source code is available on GitHub [here](https://github.com/noahcgreen/Music-Visualizer).

The demo video below demonstrates how the app works as I play the first part of
Mozart's [Sonata No. 16 in C Major (K. 545)](https://imslp.org/wiki/Piano_Sonata_No.16_in_C_major,_K.545_(Mozart,_Wolfgang_Amadeus)).

<div style="text-align: center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/P7SFjJOVkCM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>
