---

comments: true
date: 2009-10-13 16:06:27
layout: post
slug: solve-a-sliding-puzzle-with-javascript-and-your-ai-course-part-1
title: 'Solve A Sliding Puzzle With JavaScript And Your AI Course: Part 1'
wordpress_id: 456
categories:
- Development
tags:
- ai
- javascript
---

In all my years of web development and formal computer science training, you know I never really got around to truly sitting down and learning JavaScript.

Sure I knew the syntax (C based, not terribly hard), understood closures (LISP will do that to you), understood the prototype approach to Object Orientation (oddly enough playing with Python caused it to finally click).

Well [around a month ago](http://thedailywtf.com/Articles/Sliding-Around.aspx) [The Daily WTF](http://thedailywtf.com) posted one of their weekly programming puzzles.

Lo and behold, it's the classic "Eight Puzzle" that I had to solve in LISP using several different algorithms!

Feeling particular motivated I decided to finally get around to really learning JavaScript (by learning I mean really knowing the language inside and out, like I know PHP and to a lesser extent Python).

The first thing I had to tackle was: Which algorithm would I choose? There are several to choose from, DFS (depth-first search), BFS (breadth-first search), Greedy, what have you. Well, a testament to my schooling, I do remember the most optimal I could remember was the **A*** algorithm.

The A* not terribly complex once you look at it properly. Most examples discuss the A*, like most navigation algorithms, as a tree structure which, while academically correct, never actually touches a tree data structure.

The A* algorithm works by examining the current node, the possible exit directions, and how much closer the new states after exit bring you to your goal. It arranges these new nodes in a priority queue and recurses on all the new nodes until such time that the current node is the goal node.

In simpler terms: examine the paths that bring you closer to the goal first.

The Eight-Puzzle is an excellent situation in which to first play with this algorithm as all the pieces we need for the algorithm are easy to conceptualize. It's just a very simple navigation puzzle: How do we move the blank spot to it's final destination (and making sure the other pieces are set too).

By pieces we need for the A* algorithm, I mean:
	
  * **Heuristic** (or "weight" of a current path, how far are we from the goal, etc.)
  * **Expansion Function** (from our current node, what are the paths we can take?)

The Expansion function is easy. It's literally which way can you move the blank spot. If you're in the exact center of the board, the expansion function will return UP, RIGHT, DOWN, LEFT. If you're in the top left hand corner, the expansion function will return RIGHT, DOWN.

The Heuristic is a little trickier. If we knew the exact distance (number of moves) a certain state in the puzzle is to the finish, we would need a searching algorithm. Instead we estimate (hence this value being called a heuristic).

2 decent estimations would be counting the number of tiles that are not in their final position as well as calculating the [manhattan distance](http://en.wikipedia.org/wiki/Taxicab_geometry) between the blank space and it's final spot. The manhattan distance is the actual "walking" distance as opposed to the straight line distance. Think of the grid street system that is famously used in New York City. While drawing a line from point A to point B may be 2.5_ish_ miles, in reality you walk maybe 4 miles as you must walk north about 2 miles and east about 2 miles.

Even better is combining the two values. As excellent as manhattan distance sounds for an estimation, you run into inconsistencies like an unsolved board where the blank spot is in the correct position being given more weight in searching than a board 1 step away from being solved with the blank spot only 1 block away from the correct (and final) position.

In Part 2 I will step away from the algorithm and step into JavaScript to build our basic utility function (namely creating and managing a board).

**NOTE** I never finished part 2 however my code can be found [on github](https://github.com/dcousineau/javascript-learnings/tree/master/eight-puzzle) of the final solution.
