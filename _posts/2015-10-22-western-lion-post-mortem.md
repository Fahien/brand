---
toc: true
toc_sticky: true
title: Western Lion&#58; Post mortem
category: games
tags: gamejam libgdx mathematics post-mortem western
image: postfeatured.png
---

Western Lion is a collection of two minigames based on the **spaghetti western** theme developed during the one-month [Spaghetti Western Jam](https://itch.io/jam/spaghetti-western-jam) organized by [IndieVault.it](https://www.gameloop.it/).

In one of the game you impersonate *Nobody*, trying to shoot mugs before they shatter down on the pavement, while in the other game you are *Jack*, annoyed by a fly while waiting at the train station.

## Preparation

The idea of **minigames** originated from the fragmentary way scenes from spaghetti western movies I have seen over the years were coming back to my mind. Figuring out how to realize that idea was the next step.

The [libGDX](https://libgdx.badlogicgames.com/) framework was definitely a good choice from both prototyping speed and portability perspective.

Talking about graphics, I reduced it to the lowest terms as it was easier working with an 8-bit palette, borrowed from the wonderful [PICO-8](https://www.lexaloffle.com/pico-8.php) fantasy console, and a resolution of 160x90 pixels.

Audio resources were just collected by searching on [Freesound.org](https://freesound.org/).

## Jack and the Fly

The first mini-game is **Jack and the fly**. It is inspired from the concept of waiting, which has a prominent role in the beginning of Sergio Leone's [Once Upon a Time in the West](https://en.wikipedia.org/wiki/Once_Upon_a_Time_in_the_West).

Waiting makes us think, it gives us time to breath, to meditate, to observe. This emphasis projects us onto those boundless spaces and, at the same time, it allows us to get lost in the tiny details.

## Nobody is drinking

The second mini-game is **Nobody is drinking**. It is based on a scene from Tonino Valerii's [My name is Nobody](https://en.wikipedia.org/wiki/My_Name_Is_Nobody).

Nobody, the main character of the movie, bets at a particular game where you drink whiskey from a mug, then throw the mug to the air, and try to shoot at it before it shatters into pieces on the ground.

With both games key to win is **being alert and quick**. Indeed when you finish a game, time is recorded onto a leaderboard.

## Development

This is the list of tools I used to develop Western Lion:

* [Android Studio](https://developer.android.com/tools/studio/index.html),for coding;
* [The GNU Image Manipulation Program](https://www.gimp.org/) to create graphics contents;
* [Audacity](https://audacityteam.org/) to edit audio resources.

Although the right tools greatly simply the development of projects like this, a good understanding of **mathematics** and **physics** is fundamental to realize ideas that come to your mind.

* A two-dimensional **Cartesian coordinate system** helps positioning pictures at the right place on the screen;
* The **laws of motion** allow to simulate movement of objects, like the fly and the mugs.
* **Trigonometric functions** can be used to make cool smooth animations like the *sliding* in the achievements screen.

These are only some examples of the countless applications that mathematics has in the world of game development.

<div class="videowrapper"><iframe src="https://www.youtube.com/embed/D_tt83itYA8" frameborder="0" allowfullscreen></iframe></div>
