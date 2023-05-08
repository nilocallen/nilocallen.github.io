---
layout: project
title:  Orienteering A* Pathfinding
date:   2023-05-02 12:51:55 +0300
image:  /assets/images/projects/orienteering/orienteering.png
author: Colin Allen
tags:   Programming Project
---

**An optimal path finding program for a real-world orienteering course that factors in difficulty from terrain, season, and inclines using A*.**

It was made on 03.07.2021 for my introduction to AI class.

<h4>1. Introduction</h4>
This writeup details the design decisions and implementation of an A*
pathfinding algorithm specialized to generate optimal courses for
orienteering contestants during the various seasons. The algorithm is
given an image corresponding to a real-world map where pixels indicate
the terrain type, a text file detailing the elevations for each pixel, and a
path file detailing the points to travel through, in order. As output, it
generates the optimal path from point to point and displays the path and
the path’s total distance. Differing seasons enable different routes. In the
winter some ice freezes and is safe to walk on, and is therefore a potential
shortcut. In the spring, land nearby water becomes muddy and much
harder to traverse through. Lastly, in the fall, some footpaths become
covered with leaves and are harder to follow.  This is the result of the program.  The pink lines in the middle-right of the image represent the best path found by the program.
<div style="text-align: center;">
  <img src="/assets/images/projects/orienteering/orienteering-cover.png" alt="Program Output">
</div>
<h4>2. Design decisions</h4>
The algorithm will generate an optimal path for the model of the world
that it knows. Although the path generated is certainly comparable to a
real world’s optimal path, design choices such as “how much faster can one
travel through a rough meadow than a paved road” and “how much harder
is a path to follow if it is on an incline” are decided according to intuition
and not science. We will first examine how the AI determines how to
handle different terrains, followed by how to handle inclines.
<h5>2a. Terrain differences</h5>
Every terrain type, including those generated in different seasons, are
stored in a mapping of:
<div class="text-center">
{(x0 = y0), (x1 = y1), … (xn = yn)} | x = terrain type, y = a multiplier >= 1.
</div>
The easiest terrains to walk though (open land, paved road, footpath and
ice) all hold multiplier values of 1. These terrains are the base case,
representing normal human travelling speeds. This allows the difficulty of
travelling through other terrains to be thought of in terms of these base
cases, ex “it takes 1.25x as long to travel through a rough meadow than on
clear terrain.” Impassable terrains, like “out of bounds” and “impassible
vegetation,” hold an impossibly high multiplier so that they are rendered
too difficult to travel through under any circumstances.

<h5>2b. Inclines and declines</h5>
Inclines and declines are handled much more simply in the AI’s world than
in the real one. Inclines and declines have no effect on the AI’s paths other
than the increased distances that they cause. Basic internet research
reported that a 1% gradient increase will add 12-16 seconds of time per
mile travelled. With most paths being less than 6,000 m = 3.7 mi long and
having inclines that vary wildly throughout, even with this extra difficulty
taken into account the AI never made any notable pathing differences.
Additionally, there was no easily accessible information on the effect of
declines on travel speed. Considering all of this, it was concluded that the
AI’s model of the world is still consistent without the effect of inclines and
declines, and the implementation reflects this.

<h4>3. The heuristic</h4>
The heuristic is a function used to help the AI determine how close it is to a
goal state. This information is useful for identifying “promising” paths
which can be explored first, saving computation resources from being
wasted on hopeless ones. In order for the AI to generate an optimal path, it
is crucial that the heuristic be admissible. An admissible heuristic is
defined as one that never overestimates a state’s distance from the goal.
The AI’s heuristic is simply the straight line distance from a given cell to the
goal. This distance is given no multiplier for terrain.
This heuristic generates a reasonable guess for the remaining distance to a
goal in a path from any given cell. More importantly, this heuristic is
admissible. Assuming an all-powerful, flying orienteering contestant, this
distance is at worst the same as the distance the contestant still has to
travel. In reality, most paths will not involve straight line travel, making
them longer. This quality guarantees that the AI’s output is the optimal
path.

<h5>4. The cost function</h5>
The cost function is used to determine the estimated total cost of the
current path when it is completed. It is defined as
f(x) = g(start, x) + h(x, end) | g = distance from start, h = heuristic (estimated
distance to end.)
As the algorithm discovers new paths, it constantly chooses to first explore
the ones that look most promising. It identifies these paths by examining
those with the smallest cost function. The heuristic has already been
explained, so we now examine g, how the AI determines the current cost of
the path it is on.
Every cell holds a g value. Every time a cell is explored in a path, its g
value is set to be its origin cell’s g value + the cost of moving to the new cell.
This cost is determined by:
<ol>
<li>1. Getting the straight line distance between the two pixels in meters,
storing the result in variable d,</li>
<li>2. Fetching m1 and m2 = the terrain multipliers for each pixel, and</li>
<li>3. Returning (½ * d * m1) + (½ * d * m2)</li>
</ol>
Step 1 is simply a re-use of the heuristic function. Step 2 and 3 are how the
AI knows to avoid rugged terrains having higher distance multipliers; the
cost for going from cell to cell in these terrains is much higher than
traversing a terrain with a lower multiplier, so the AI will avoid difficult
terrains unless it either has no choice or there is no alternative that saves
time. In most cases, m1 = m2. Step 3 is designed as it is to account for
movement into new terrain types.

<h4>5. Seasonal algorithms</h4>
Differing seasons have differences that affect their pathing. Summer has
no changes. Each season is operated on by the same AI with no changes to
the features described earlier. However, the map that it searches through
undergoes seasonal changes, resulting in a different path. The map is
stored as a two dimensional array, with map[x][y] = Cell. Each cell stores
information about that pixel, like terrain type and elevation. Each season
modifies the terrain type of certain cells before searching for a path. For
example, some water cells in the winter are frozen to make ice that can be
walked over.
Note: The only seasonal differences are those described. Implications any
season may have on terrain multipliers are not accounted for. This means
that the AI does not expect snow in the winter to slow it down, nor rain in
the spring, etc.

<h5>5a. Fall</h5>
In the fall, the terrain type “easy movement forest” experiences heavy leaf
fall. This makes it harder to follow any footpaths running through or
adjacent to these terrains, as they are covered up by leaves. To account for
this, the AI scans every cell in the map. If it finds a cell of terrain type
footpath, it scans every cell immediately surrounding it. If any of the
surrounding cells are of terrain type “easy movement forest,” it converts
the footpath cell into a cell with a special terrain type “leaf covered path.”
The leaf covered path terrain type has a 1.1 cost multiplier, compared to a
footpath’s normal 1. This results in the AI seeking out footpaths less often
in the fall, as they are more costly to travel through.
<div style="text-align: center;">
  <img src="/assets/images/projects/orienteering/fall-code.png" alt="Code for fall weather">
</div>

<h5>5b. Winter</h5>
In the winter, water cells within 7 cells of non-water cells become ice that
is safe to walk on. The AI achieves this by first scanning through the whole
map and adding every edge of every body of water to a list called
shorelines, and converting it all to ice. This list is then iterated through 6
times. Each iteration, every water-ice edge from every element of
shorelines is added to a list called new_shorelines and turned into ice. After
shorelines is exhausted, shorelines is set to new_shorelines, and the cycle
repeats until every water pixel within 7 pixels of land is turned into ice.
<div style="text-align: center;">
  <img src="/assets/images/projects/orienteering/winter-code.png" alt="Code for winter weather">
</div>
    
<h5>5c. Spring</h5>
In the spring, any cell within fifteen cells of water that can be reached from
a water cell without gaining more than one meter of elevation across cells
are now underwater/mud. This changes their terrain multiplier to 8,
massively slowing down travel.
To do this, the AI begins by scanning the entire map for water edges as it
did for winter. These edges are added to a list called waters. As before,
waters is iterated through 15 times. Each iteration, for every water cell in
waters, every neighboring land cell’s elevation is checked. If the difference
in elevations from the land cell to the water cell that made it is less than 1
meter, the land cell is added to a list called mud and then turned into mud.
After the waters list is emptied, waters is set to mud and the process repeats
until all cells within 15 cells of water have been examined and transformed
accordingly
<div style="text-align: center;">
  <img src="/assets/images/projects/orienteering/spring-code.png" alt="Code for spring weather">
</div>