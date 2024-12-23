---
layout: post
title:  "Throwback: Conway's Game Of Life Explained"
date:   2024-10-19 00:19:13 +0100
categories: life
---

<b><big>
<u>Preface</u><br>This post is from 2018, I've moved it to this blog but the original can be found at <a href="https://github.com/game-of-life-in-X/The-basics-of-Conways-Game-of-Life/tree/master">https://github.com/game-of-life-in-X/The-basics-of-Conways-Game-of-Life/tree/master</a><br><br>It mentions a surprisingly cool Python program I wrote that can be found <a href="https://github.com/game-of-life-in-X/GameOfLifeInPython">here</a>, a preview image is below (if it doesn't play in your browser it can be viewed <a href="https://raw.githubusercontent.com/game-of-life-in-X/GameOfLifeInPython/refs/heads/master/models/glider/preview.gif">here</a>):</big></b>

<img src="https://raw.githubusercontent.com/game-of-life-in-X/GameOfLifeInPython/refs/heads/master/models/glider/preview.gif" style="border: 5px solid #555">

*The original post is below:*

<hr><br>

# The basics of Conway's Game of Life

![](https://camo.githubusercontent.com/730adf400902d41499463f0e5d3d73415bf2dd80e229b62322bdbc9a258911b1/68747470733a2f2f692e696d6775722e636f6d2f624945306a6c752e706e67)

## What is it?
Conway's Game of Life is a cellular automaton designed by the British mathematician John Conway in 1970. The player creates an initial state for the game 'grid' and then provides no further input and watches how the cells 'evolve'. The model is an example of the concept of complexity derived from simplicity as the game holds an extremely simple set of rules that are then used to create more complex models and ideas.

## How can I try it out?
You can use my own [python implementation](https://github.com/game-of-life-in-X/GameOfLifeInPython) or the common cross-platform implementation [Golly](http://golly.sourceforge.net/). There are also online alternatives if you want to avoid installing a program and performing setup.

## The rules
The rules for Conway's Game of Life are extremely simple; Each cell is in a binary state of either dead or alive (usually with black representing a alive and white representing dead) and cells both alive and dead interact with neighbouring cells to produce an output, either changing state or remaining in the same previous state. Cells count as neighbouring if they are horizontally,  vertically or diagonally adjacent to the cell. The game moves in 'ticks', with each tick acting as  check in state for every cell on the grid. There are four rules that dictate what happens to a cell each tick:
1. Any cell with less than two neighbours will die. This is known as underpopulation or loneliness.
2. Any living cell with two to three neighbours survives to the next generation.
3. Any living cell with greater than three neighbours dies . This is known as overpopulation or overcrowdedness.
4. Any dead cell with three neighbours will become a live cell. This is known as reproduction.

## Basic patterns
The most common types of patterns come in three forms:
- Still lifes which are static cell groups that never change
- Oscillators which change through a set of looping forms in an infinite loop, with each change being known as a period 
- Spaceships or Gliders which are moving groups of cells that move forever unless disrupted. 

### The block
![A block model](https://camo.githubusercontent.com/66039e85df856a541caab034090868a2811dfd9f7b1ba1beb662be91516b1577/68747470733a2f2f692e696d6775722e636f6d2f647667794961742e706e67)


The block is the simplest still life and is extremely easy to create. It is often left as the result of more complex models.

### The hive
![A basic hive model](https://camo.githubusercontent.com/fa2b4bd14e898186370d0d166abd91df2a56e45708e70b27a741ec968467cdea/68747470733a2f2f692e696d6775722e636f6d2f4c524c426769452e706e67)


The hive is another simplistic Still Life model. It useful for the creation of some more complex models.

### The Blinker
![The blinker](https://camo.githubusercontent.com/6d945c9bad7dcdd69668b4e4568465cc38a1556959cf47e6320f73cfa639d1d5/68747470733a2f2f692e696d6775722e636f6d2f7358695235594d2e676966)

The Blinker is a simple two period oscillator, it stays in place switching between the two periods forever.

### The Toad
![The toad](https://camo.githubusercontent.com/6b41cddae01025d203de7e7d39699bb35a4499d6a4a111c8c280784e609b59a5/68747470733a2f2f692e696d6775722e636f6d2f645238345258662e676966)


The Toad is another two period oscillator. It's slightly more complex and has a further reach than that of the blinker.

### The Glider
![A basic glider](https://camo.githubusercontent.com/fb190dc024f02f5e5e4c45e3c4de10fa74d2065f4a00510d06099e68ff9335d1/68747470733a2f2f692e696d6775722e636f6d2f726e506f6837712e676966)


This is a classical glider, it moves infinitely in one direction and doesn't stop until it meets an external cell.

### The Eater / Fish Hook
![The most efficient eater form, known as a fish hook](https://camo.githubusercontent.com/496b19b1280c5b14e5c54c51d7cfb8f31d430b2419682628448fb28ecf71ec33/68747470733a2f2f692e696d6775722e636f6d2f77376a74556f362e706e67)

A specialised Still Life model that can consume oncoming gliders. It is useful for more complex mechanisms such as logic gates.
*Note: This model may not work on certain algorithms*

## Where to go next?
From here it's up to you to experiment and research to find more complex and interesting models and create new ideas. Youtube user [Alex Bellos](https://www.youtube.com/channel/UCjY-JWyBWiejeMoVSmYVTPA) has a playlist featuring many more complex models such as glider guns and basic logic gates using gliders that you can explore [here](https://www.youtube.com/watch?v=bTPN3spiq1I&list=PL_DEGJtvl7wtPc-ZyTq_jh0ptRjnYGaWZ). There is also the [Life Wiki](http://www.conwaylife.com/wiki/Main_Page) which is extremely helpful for understanding more complex models and ideas in Conway's Game of Life. 
