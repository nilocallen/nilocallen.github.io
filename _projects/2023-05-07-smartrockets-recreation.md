---
layout: project
title:  SmartRockets Recreation
date:   2023-05-07 14:45:55 +0300
image:  /assets/images/projects/smartrockets/smartrockets.png
author: Colin Allen
tags:   Programming Project
---

**A recreation of SmartRockets, as popularized by <a href="https://www.youtube.com/@TheCodingTrain">TheCodingTrain</a> on YouTube.  It was created with the p5 JavaScript library using Processing, and references code from TheCodingTrain often.**

SmartRockets is a popular project for learning genetic algorithms.  It involves a spawn point at the bottom, a rectangular obstacle in the middle, and a target at the top.  Generations of rockets spawn from the spawn point, then attempt to reach the target within a time limit or die trying.  Once time's up, the DNA of every rocket from the generation is combined into a new population of rockets, with the best performers' genes being most present.  This means that the later generations have a large percent of rockets hitting the target.    

This project got me comfortable with genetic algorithms, and here I'll pass on what I've learned in the context of SmartRockets.  

<h4>The Rockets</h4>
The agents in this problem are the rockets.  The population is made from groups of agents, or rockets.  Each rocket has DNA, which it uses to interact with the world around it.

<h5>Rockets: Fitness</h5>
The most critical data attached to the rockets are its DNA and fitness.  The fitness value is a normalized value (0 - 1) that indicates how well it performs the task of reaching the target.  The fitness is calculated at the end of each population's attempts to reach the target as follows: 
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/fitness-code.png" alt="Fitness code">
</div>
The first estimation at fitness is determined simply by the straight line distance from where the rocket ended to the target.  Then, the fitness is lowered or raised according to the rocket's performance.  For example, crashing at the bottom of the screen is a much larger failure than crashing at the top, side, or into the obstacle.

<h5>Rockets: DNA</h5>
The rockets' DNA determines how it moves.  For this project, the DNA is simply an array of force vectors of length *n* where *n* is the amount of frames the simulation runs for.  Each frame, the force vector at index *i* is applied to the rocket, where *i* is the current frame count.  At first, this DNA is chosen randomly.  The changes to this DNA are detailed later during the discussion of the population.  

<h4>The Population</h4>
The population is the group of every rocket in a generation, and is where the evolution is managed.  All members of the population move according to their DNA, then when time's up, the key parts of the genetic algorithm are executed.  These parts are the evaluation function and the selection function.  The selection function also handles the genetic crossovers.

<h5>The Evaluation Function</h5>
The evaluation function is responsible for evaluation each rocket and creating the mating pool for the next generation based off of those evaluations.  This function is why better rockets are represented more in future gene pools.  To achieve this, the function creates an array of rockets for the mating pool.  Then, it adds each rocket from the previous generation to the mating pool.  The key here is that rockets with higher fitnesses are added significantly more times.  This means that later, when the mating pool is used to create the next generation of rockets, the parents that are chosen are significantly more likely to have been rockets with high fitness scores.  
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/evaluation-code.png" alt="Evaluation code">
</div>

<h5>The Selection Function</h5>
The selection function is much simpler, and is responsible for choosing parents from the mating pool and creating the new generation from their genes.  
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/selection-code.png" alt="Selection code">
</div>
To create the genes of the child, we use a crossover function:
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/crossover-code.png" alt="Crossover code">
</div>
This function chooses a random midpoint in the genes array and pulls genes from the beginning to the midpoint from parent 1 and from the midpoint to the end from parent 2.  

To prevent the rockets from being stuck on local maximums, the genes have a random chance to mutate.  
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/mutation-code.png" alt="Mutation code">
</div>
Every now and then a random gene will be thrown back into the array to encourage diversity.

<h4>Results</h4>
This genetic algorithm accomplishes its goal, but with a low success rate.  Here is the result after 1000 generations:
<div style="text-align: center;">
  <img src="/assets/images/projects/smartrockets/finished-run.png" alt="Finished run ">
</div>
The population never converges on an ideal solution for most of the rockets.  

<h4>Shortcomings of Design Choices</h4>
Genetic algorithms are extremely flexible and can be implemented in many ways.  For this project, I often chose the simplest implementation of a feature so I could move quicker in my learning.  Here are some of these choices, their ramifications, and what I will change if I ever revisit this project.

<h5>Parameters</h5>
Genetic algorithms have many parameters, and the outcomes of changing them are often complicated.  There is much research on how these parameters impact the simulation that are outside the scope of this project, but I encourage you to read about it if you're curious.  The parameters of this algorithm are extremely unoptimized.  Most were chosen arbitrarily and tweaked until a remotely successful result was shown.  As a result, the simulation never reaches a population with greater than around a 10% success rate.  These parameters include:
<ul>
  <li>Population size - the simulation was only ran with a population of 100, potentially limiting the solutions that are found.</li>
  <li>The fitness function - rewards and penalties to fitness in response to certain events are arbitrarily chosen.  This means my rockets are likely often penalized to greatly for their failures and rewarded too highly for their successes.</li>
  <li>The evaluation function - the most fit rockets are added 200 times at a maximum, with less fit ones being added anywhere from 0 - 199 times.  This could be partially why the population never converges to greatness, as too many unfit rockets are added to the pool.</li>
  <li>The crossover function - genes are combined from a randomly chosen midpoint.  The crossover function in genetic algorithms has a great effect on performance and maybe a different method would be more fruitful.</li>
</ul>

<h5>General Properties</h5>
This genetic algorithm has some properties that hold it back.  These include:
<ul>
  <li>Lack of generational protection for the most fit - if a rocket contains the DNA to reach the target, but then has an unfavorable crossover or mutation,  its strong genes can be lost into the next generation.  Because of this, performance often fluctuates from 10+ successful rockets to 0-3 in the next generation, and progress has to restart.  This can be prevented using a different selection function, or ensuring that the best rockets of a generation are in the next generation's pool. </li>
  <li>Tendency to converge on local maximums - once a rocket finds a solution, most other rockets will emulate this solution exactly.  So, if a rocket finds a slow or winding solution, other rockets will mimic this slow or windy solution instead of finding a faster one.  And, they will almost always all go around the obstacle in the same path.  A higher mutation rate could help to combat this, but with our current DNA structure, I doubt this could be truly fixed.</li>
</ul>

<h5>Upon reflection,</h5>
This project taught me a lot about genetic algorithms, what affects their performance, and why.  It contributed toward my passion for AI in programming, and helped me figure out how I want to fit into the industry professionally.  

Thanks for the read!