+++
title = "New Project: Conway's Game of Life"
date = 2024-08-25T17:21:50-07:00
draft = true
+++

## Beginning with First Concepts

Conway's game of life is a well known [cellular automation](https://en.wikipedia.org/wiki/Cellular_automaton) which takes place on a lattice grid. The rules for generating each step of the simulation are the following:

- Any live cell with fewer than two live neighbours dies, as if by underpopulation.
- Any live cell with two or three live neighbours lives on to the next generation.
- Any live cell with more than three live neighbours dies, as if by overpopulation.
- Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

The simulation begins with a seed which evolves over time. Seed behavior is subject to the [halting problem](https://en.wikipedia.org/wiki/Halting_problem), which in this particular case means that the final outcome of the seed cannot be predetermined by a generalized algorithm. We can, however, make a couple of assumptions about how the simulation will behave. The simulation will commonly devolve into a combination of the following features:

1. Simple stable structures which sometimes are completely static and sometimes move within a small radius.
2. Stable moving structures which retain a similar shape as they move along the grid in a fixed direction.
3. Stable structures which emit structures as specified in 2.
4. A combination of 2 and 3.

The main shared idea between these different features is that they are stable, so they behave in predictable ways. Being turing complete means that there are really an infinite number of these "stable" structures which defy categorization, so these categories are just a basic communication of what one can expect.

## Project Goals

My aim is to implement this in Java using the Swing window API and the Graphics2D class. In order to have a reasonably usable application, I want a few basic features:

- A working simulation which is relatively performant.
- Panning around a grid by clicking and dragging the mouse. Zooming with the mouse wheel.
- Ablility to pause/play the simulation using buttons on a menu bar and change the simulation speed.
- Being able to click on cells to turn them on or off.

These features are the baseline of what I would call usable if I were to compare it to other solutions avaliable on the internet. There are more features I am interested in adding at some point:

- Compatibility with the many formats of seed files used for loading and saving the current state of the simulation.
- More editing tools such as a pencil/eraser, a marquee/lasso for changing large areas at once, a clipboard for copying/pasting commonly used structures, etc.
- Statistics on cell population change over time
- Color theme creator
- Cell history which shows "shadow" in cells which have been on in the past.
- Ability to automatically follow moving structures

Most of these features are relatively easy to implement, but will take some doing. 
