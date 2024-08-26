+++
title = 'Implementing the Grid'
date = 2024-08-25T18:16:49-07:00
draft = true
+++

## Memory is Real

In a theoretically pure version of Conway's Game of Life, the grid extends towards infinity in all directions. In cushy math world, this is [fine](https://en.wikipedia.org/wiki/Infinite_set). In implementing such an idea in a computer though, we are presented with an engineering problem. How does one create an infinite space with only a finite amount of memory to store it in?

### Static Array

The first and simplest way of doing this is to simply not. We can create a two-dimensional static array of cells, all instances of a Cell class. Each cell keeps a list of eight neighbors which are directly adjacent to it. To update the simulation we iterate over the whole array having each cell check its neighbors and decide whether it should be on or off in the next step, then iterate over the array again to update all of the cells. Cells on the edge of the array which have less than eight neighbors simply treat the non-existent neighbors as being off for puposes of counting surrounding cells which are on. Cells which would attempt to appear outside of the array are ignored.

This, suprisingly, is the solution on this [popular site](https://conwaylife.com/) which offers a JS implementation of the game. I find this solution deeply unsatisfying and not much of a solution of all. We can do better.

### Toroidal Static Array 

This approach is very similar to the last, except that instead of disregarding the edges of the array, we skirt around them by cleverly assigning the neighbors of cells on the edges. We will assign neighbors to each cell as normal, except the edge cells will wrap around to the other side of the array. This way we have an effectively infinite amout of space.

![](/toroidal.gif)

This approach works for a simple implementation, but has some issues with being equivalent to the infinite grid. As a basic example: since a moving structure can loop around, it could interact again with static structures in a way that it wouldn't on the infinite grid. For this project, I decided that this approach was not satisfactory.

### Dynamic Array Allocation

My next idea was to dynamically allocate cells in ArrayLists representing the quadrants on the cartesian plane as the size of the simulation grew larger. This way, the grid would grow from the center, and cells could be inserted at amortized *O(1)* time.

I used an enum here, Quadrant, to handle how each quadrant's coordinates were assigned. With four relative arrays, the cells needed some way to know what quadrant they were located in. The *getXParity()* and *getYParity()* methods in quadrant both give relative parities of the cartesian quadrants, which is multiplied with the cell location to draw the cells within the quadrants they belong to.

![](/dynamic.png)

The plan with this solution was to grow the array only when an on cell came close to the edge of the current array in memory. This plan was flawed however. Before the simulation was even implemented I attempted to initialize the array with large values for *INIT_AMOUNT* to check for any issues. I found that not only did heap run out of memory at a relatively small value, panning the view around became worse and worse. Whenever the view was panned with a large amount of cells, the quadrants would "detach" from each other probably due to some multithreading desync. I attempted to limit the number of cells being drawn at once but the performance was still terrible. This idea was just **not scaleable** at *O(n²)* memory complexity. So what is?

### Hash Registry and Relative Grid

Since I was using too much memory tracking all the cells, a more elegant solution would be something with a lot less memory complexity. I had another idea which I held off implementing to avoid premature optimization. Most of the cells which the previous solution stored in memory were not doing anything at all. Really, the only cells which mattered for each simulation step were those which were on and those which were adjacent to the ones which are on. Anything else can be safely assumed to be completely blank, and not necessary to store in memory.

The current solution uses a cell registry, where every cell created is inserted into a hash table with the key being the cell's location on the grid. To check for a cell's neighbors, it is easy enough to access the HashTable at *O(1)* for each neighbor. This means that updating the cells only takes *O(n)* time and storing the cells takes only *O(n)* memory. Even better, *n* refers to the number of tracked cells rather than the dimensions of the whole array, which means that *n* will always be less than or equal to *n* in the previous solution.

![](/hash.png)

The Direction enum is similar to the Quadrant enum in functionality, but baked a little differently due to the difference in approach. It deals with direction relative to a cell, not quadrants relative to each other.

This does leave one issue though. The cells which are no longer in memory formed the grid before, so now there's no grid display. This fix was much easier than I expected.

![](/grid.png)

By measuring the window size, it's easy enough to render a grid of squares that is the size of the window, which is flexible enough to work with any window size. Then in order to simulate motion, it can be moved relative to the offset that the tracked cells use to pan around. Applying modulo to the offset allows an infinite grid to be simulated very easily.

## Limitations

So how infinite is it *really?* Well, the main limiting factor is the integer limit. Because the Java Graphics2D class only accepts integers, once either the pan offset or one of the cell coordinates surpasses the integer limit, there isn't really a way to draw more. Java does support arbitrarily large numbers through the BigInteger class but it doesn't seem worth it to find a graphics API which allows the use of this, as the integer limit is already ±2,147,483,647. Still, it would be good to make sure there is no crash in the case of a structure reaching the edge.