![screenshort](https://github.com/fastrgv/hbox4/blob/main/t7s.png)

Here is a link to all source code and build files:

https://github.com/fastrgv/hbox/releases/download/v1.1.9/hb24dec24.7z

Type "7z x filename.7z" to extract the archive.

* On OSX, Keka works well for 7Z files. The command-line for Keka is:
	* /Applications/Keka.app/Contents/MacOS/Keka --cli 7z x (filename.7z)




#### Note to github downloaders: (Please ignore the "Source code" zip & tar.gz files, which are auto-generated by GitHub). Download the latest release by clicking on the large 7z file under releases. It contains all assets for Windows & Linux, including source code. Type "7z x filename" to extract the archive. Then you can compile your own binaries, or use the pre-built ones provided.










Alternate download link ( "4" is not a typo ):
	https://sourceforge.net/projects/hbox4/

# hbox5 -- sokoban solver using Ada


#### What's new:



**ver 1.1.9 -- 24dec2024**

* Fixed embedded version code that is used in my RufasSok, Sokerban, SliderPuzzles apps.
* Added a 5th commandline parameter: TimeOutSec (integer). When omitted the default fallback is 660 seconds.
* Reduced memory-release-delay at end of external solver execution.


**ver 1.1.8 -- 13dec2024**

* Added a memory check to assure its availability: Windows, OSX, linux.
* Enhanced portability of my linux build [by using an old compiler].
* Delayed the endgame, thus giving more time for the full set of heuristics to be in play.
* Added a 5th heuristic that generally improves performance; hence the name change.
* Bumped default timeout limit from 10 to 11 minutes; but, as always, (ctrl)-c quits at any time.


**ver 1.1.7 -- 2jan2024**

* Fixed potential indexing error that adversely affected efficiency.


**ver 1.1.6 -- 5dec2023**

* Added more tweaks for faster execution.


**ver 1.1.5 -- 1dec2023**

* Reverted to older & faster splay queue implementation.
* Revised /docs/xsok90times.txt (smaller average).


**ver 1.1.4 -- 25nov2023**

* Now use preprocessing to determine minimal valid and "live" box positions.
* Extended box-count limitation from 24 to 32.
* Began the rigorous enforcement of theoretical limitations: 32 boxes, 256 live box positions.
* Restructured data to better conserve memory usage without impacting runtimes.

#### More change-history at end of this file



## Description

This is a commandline-terminal sokoban solver written in Ada. It is "generic" in the sense that it contains no domain specific strategies. It also provides a demonstration of the advantage in using the Hungarian Algorithm.

-----------------------------------------------------------
Featuring

	* no installation
	* no dependencies
	* simply unzip in your Downloads directory, and run.
-----------------------------------------------------------

Pre-built executables are provided in 3 variants:

	* hbox5.exe (Win64, including Windows 10 & 11)
	* hbox5_gnu (linux)
	* hbox5_osx (Mac/OSX)


Note that this solver may be run from a thumb drive: Simply unzip onto a USB flash drive formatted to match your system, and run.


## Setup

Windows users please read ~/docs/windows-setup.txt for details.
For OSX see ~/docs/osx-setup.txt. 
Linux users can probably figure it out.

But generally, you should first open a commandline terminal window.
Then use the 7zip command to extract the archive and maintain the directory structure:

	7z x <file-name>.7z [this unzips contents to a new game-dir]

	cd <game-dir>

Then issue the proper commandline syntax to start the app.



## Usage

Sokoban puzzles typically come in files containing groups of puzzles, so the user interface assumes that you provide a file-name, and 1 integer representing the number of the puzzle to be solved. The solver name is "hbox5".

EG: hbox5 games/Sladkey.sok 22

is the command to solve level 22 from the file "Sladkey.sok". The solution is written as a long string in the terminal window at the end of processing. Of course it could be redirected to a file thusly:

EG: hbox5 games/Sladkey.sok 22 > soln.txt

-------------------------------------------------------------
In addition to the 2 mandatory commandline parameters discussed above, there are 3 more optional ones:

* (3) [float] MaxGb memory to use
* (4) [int 0..3] Solution method:
	* 0 (default) "quickest" solution [a cfg updated only if #pushes is reduced]
	* 1 move-reducing CFG-updates [a cfg updated even if #pushes is equal but #moves is reduced]
	* 2 No Hungarian Estimator: possibly more move-efficient solutions but typically much slower.
	* 3 Like method 0 but also penalizes total-moves [ p0= 0.9(p0) + 0.1(moves/5) ].
	* 4 Like method 1 but also penalizes total-moves [ p0= 0.9(p0) + 0.1(moves/5) ].
	* 5 Like method 2 but also penalizes total-moves [ p0= 0.9(p0) + 0.1(moves/5) ].

	* 10..15 triggers the single priority hbox1 "baseline" option for the above 6 methods [not for normal use].

* (5) [integer] TimeoutSec

EG: hbox5 games/Sladkey.sok 22 6.5 1 sladkey22.txt

indicates using no more than 6.5 Gb of memory and the efficient! solution method AND to write the output to a file in the current directory named "sladkey22.txt". !But note that the solutions are not always efficient. This name merely refers to certain algorithmic choices within the solver which generally tend to produce solutions with fewer moves.

EG: hbox5 games/Sladkey.sok 22

indicates using the defaults, i.e. 6.5Gb memory and fastest solution.

EG: hbox5 games/Sladkey.sok 22 5.5 10

indicates method 0 but using "baseline" single priority measure for comparison purposes.

There are many puzzles this algorithm will not solve due to memory limits, so the embedded memory limiter will exit gracefully when memory usage exceeds the preset limit. 

Finally, if you don't want to wait for the solver to finish, you can (ctrl)-c out of it to quit immediately.


## Algorithm Used

A multiple-heuristic, single-step box search, done in **reverse**, using five "orthogonal" priority measures [heuristics] in a round-robin sequence. The Hungarian Algorithm is used to match boxes with goals and to generate an estimate of future box moves to solution.

An article by Frank Takes shows advantages to working from a solved position backwards to the start position. This prevents box-deadlocks from taking up space in the search tree. Thusly, the formidable issue of deadlock avoidance is completely ignored. Likewise, the tricky issue of endgame goal-packing-order is sidestepped, as well.

Self balancing splaytrees are used to test whether a given configuration was seen before. There are also 5 priority queues embedded into the splaytrees that provide distinct "views" of the data.

The first 4 priority measures were adapted from the "Festival" algorithm description, but possibly simplified to allow rapid local evaluations:

* pri1: Nboxes - NboxesOnGoals...............0 means all boxes on goals
* pri2: Ncorrals - 1				...............0 means only 1 corral, i.e. pusher is free to roam
* pri3: NblockedRooms			...............0 means no boxes block openings
* pri4: NblockedBoxes			...............0 means all boxes accessible to pusher

* pri5: Nfar						...............non-linearly drives most-distant boxes nearer their goals

Note that priority measure #1 counts a box as being on a goal only if it lies on its Hungarian-assigned goal, except when the non-Hungarian solution method is used.

These 5[primary] priority measures are typically small non-negative integers to be minimized. So it often occurs that there are many candidate nodes of the search-tree with the same measure. 

So to help distinguish those that are more promising, a secondary priority measure [pri0] is used that counts the total box-pulls to reach the current configuration, **PLUS** the Hungarian-Algorithm-based estimate of the remaining number of pulls. I.E.

* pri0 := pulls + HungEst

which currently has a range limit of 0..700.

If the puller is blocked [by boxes] from reaching its goal, then one is added to "pri0" as a penalty.

-------------------------------------------------------------------------------
#### skippable detail
* The newest methods #[3,4,5], the secondary measure (pri0) also considers total moves as 10% of cost; I.E.
	* pri0 := 0.9(pulls+HungEst) + 0.1(moves/5)
* where the arithmetic is simply to stay within the current range limits of 0..700. 
* Note that #moves is typically about 5 x #pulls; hence the magic number 5 used to equilibrate measures.
------------------------------------------------------------------------------

Finally, the round robin regimen that includes all 5 measures eventually drops the last 4 measures somewhere beyond the halfway point since corrals and blocked rooms must be permitted at the end of the reverse game, i.e. near the beginning of a forward game, and because the evaluation of the measures is costly.

### Additional algorithmic details

A nodal configuration description consists of the box layout, the upperleft corner of the puller-corral, as well as the total box pulls, the hungarian estimate of the future box pulls, and the 5 [primary] priority measures.

Conceptually, the Splaytree Priority Queue {frontier} contains 5 independent queues, each ordered by one of the 5 priority measures mentioned above. So if the round-robin parameter is K in {0,1,2,3}, we greedily pop the leading candidate node off of the front of queue #K, thereby deleting it from {frontier}.

So, in this SingleStepBoxSearch the initial configuration is read, evaluated and pushed into the {frontier} queue. But goal and boxes are interchanged because this is a backward search using a "puller" rather than a "pusher".

So here begins the description of the main loop:-------------------------------------------

Choose "K", and pop the frontrunner off Kth queue and look at its nodal information to retrieve the hashkey. This key allows direct access via the splaytree structure that, in turn allows direct removal of the node from the splaytree and the other 3 PriQueues. Thusly, the removals from the other 3 PriQueues do not require any time consuming list-traversals.

This removed configuration is then tested to see whether a solution has been reached. If not, it is inserted into the {explored} splaytree, which has no queue structures attached. The splaytree allows rapid insertion and rapid random retrieval in case we find a solution and need to reconstruct our path. 

For this current configuration we simply cycle through each [unsorted] box and try to move it one step in each direction. If the move is successful, its 5 priority measures are evaluated and it is enqueued into {frontier}. This involves a splatree insertion and 5 priority-queue insertions that each maintain a separate ordering. 

End of main loop.------------------------------------------------------

Addendum1: after exchanging emails with Yaron Shoham, author of Festival, I have come to believe that Pri#4=NblockeBoxes is not quite "orthogonal" to Pri#3=NblockedRooms, meaning that #4 may occasionally enhance the effectiveness of #3 (and in fact, seems to). This would also imply that #4 could be omitted, as Yaron has proposed.

Addendum2: the newly added 5th priority measure is described as follows:

	* Nfar = 60(bmx0-bmx)/bmx0

(the magic number 60 is the maximum numerical value allowed for priority measures 1 thru 5)
where bmx is the sum of the squares of the box distances to their hungarian-matching goal.

The desired behavior, in the forward game, is to clear out the boxes closest to their targets first, so subsequent moves have fewer obstacles. Thus, in the reverse game, we want to motivate placing the boxes furthest from their goal **before** we address the boxes relatively close to their goal.
You might observe that there are already heuristics that drive boxes toward their goals. That is true, but they are linear in their effect. The intent here is to create a nonlinear effect that would prioritize the most distant traversals. My motivating example was problem #4 from the classic set of 90, which had been unsolved by hbox until now.


## What's so great about this app?

This is only a moderately capable sokoban solver (solving 45 of the original 90). What makes it interesting? In a world with extremely capable solvers like Sokolution and Festival, why look at this one? Why hunt with an algorithmic crossbow instead of a gun? Simplicity & elegance. hbox5:

* contains no domain-specific specializations or strategies [other than the 5 priority measures]

* uses algorithms and data structures of general interest and usefullness.

* demonstrates the considerable power of the Hungarian Algorithm to help direct effort.

* easily buildable on Windows, OSX, and Linux using free GNU Ada compiler.

It also adds further proof of the effectiveness of the design choices made in the Festival solver that proposed the original 4 "orthogonal" heuristic measures, or so-called "features".


### SplayTree-Priority-Queue
The splaytree [self-balancing-binary-tree] based priority queues allow unique hash keys to be inserted, found & removed quickly, with 5 embedded priority queues that allows efficient insertions, access, and deletion, both from the heads [popping], and directly by key. The hash keys uniquely identify pusher & box-layouts at each saved node, to avoid duplicates. The 5 priority queue orderings allow primary and secondary (tie-breaker) priority measures. As mentioned above, the structure also allows rapid, direct access deletions given the hashkey [O(log n)].

The only expensive [O(n)] operation is finding the head of each queue. That involves a lexicographical search through a 2 dimensional array of pointers to find the first non-null pointer. All other queue operations are done in constant time [O(1)], i.e. independent of the number of queue entries.

It was very difficult to get these priority-queue operations to be efficient enough for use in a sokoban solver. The speed of this quintuple-queue data structure and algorithm is really quite remarkable!

### Dynamic Programming [flood-fill]
Dynamic programming allows efficient determination of box-valid locations and the feasibility and cost of traversing between two locations. This information is used to feed into the Hungarian Algorithm.

### Hungarian Matching Algorithm
This fully functional implementation of the Hungarian algorithm provides valid, one-to-one pairings between boxes and goals for each intermediate box-layout, and provides realistic estimates [admissible & consistent] of the number of box-moves remaining to completion. The power of this heuristic can be seen when it is swapped out; i.e. when the secondary priority measure is changed from (moves + estimated-future-moves) to just (moves), the solver is still functional, and even faster on a some puzzles, but is significantly impaired on many others.

* admissible means: heuristic(x,goal) <= actualCost(x,goal)
* consistent[monotone] means: h(x,g) <= actualCost(x,y)+h(y,g)

Note that consistent => admissible assuming h(g,g)=0.

One may choose to omit the Hungarian estimator so that the secondary [tie-breaking] priority measure then becomes simply the total box moves used to arrive at a given configuration.

#### Note on the Hungarian Algorithm: 
I have searched for a correct version online, but found none. I found several that "almost" worked but were all flawed, mainly, I think, due to the age and nature of the original algorithmic description. It was invented before computers were widely available, so was described in terms of hand computations, parts of which are quite confusing, possibly due to language ambiguities. [Kuhn, 1955]

The algorithm used here was copied on 20sep18 from: https://users.cs.duke.edu/~brd/Teaching/Bio/asmb/current/Handouts/munkres.html [now dead link] and modified to correct some errors.


### Simplicity & Generality
These qualities result from a minimalistic regimen that AVOIDS:

* complex control mechanisms, high or low level, abstract or detailed;
* domain-specific improvements, tactics;
* databases;
* macro-moves, tunnel-macros, goal-macros;
* matching specific box-pattern-templates;

For the C++ programmer this Ada code is written in a transparent style and should be easy to comprehend; and for the experienced Ada programmer there are many improvements to be made to better utilize the advanced protections and features of the Ada language.  


## Shortcomings

Currently handles only 32 boxes or less.

Disclaimer #1: the elegance lies in the algorithms, not the code.

Disclaimer #2: No attempt at solution optimality is made. The goal in this solver is to find any solution. The 5 orthogonal "features" do not lend themselves to finding solutions with any type of optimality.

Disclaimer #3: even using the option for a more efficient solution, rather than the quickest, there are strange moves produced that are clearly wasteful. Still, the solver is somewhat surprising in its capability. I have been trying for years to solve level #2 of the original 50 with the usual breadth-first search methods, but "hbox5" solves it easily. Another surprise is that it solves takaken #7 ("love") in 3 seconds, while the excellent takaken solver that I downloaded takes 27 seconds [on the same computer]. And, assuming I ran it correctly, Festival took 283 seconds!

In any case, I wish to expose this algorithm to public scrutiny, and allow anyone with an interest, the chance to improve or extend this generic approach to the branch of artificial intelligence that addresses the formidable task of solving sokoban puzzles.


## Xsokoban Levels Solved (updated late 2024):

  1  2  3  4  5  6  7  8  9 12 
 13 15 17 18 29 33 37 38 41 43 
 44 49 51 53 54 55 56 57 58 59 
 60 62 64 65 68 70 71 78 79 81 
 82 83 84 86 87

for a current total count of 45 for hbox5.

Hbox1 may add some to this total, where
hbox1 refers to the methods numbered {10,11,12,13,14,15}
that use a single priority measure: BOG [boxes on goals]
rather than 5.

hbox5 is extravagant with memory; all failures
I have seen are due to shortage of memory, & lack of progress.


## Build Instructions:

Three usable, pre-compiled binary executables are delivered (Windows/Linux/OSX). But if you choose to try to rebuild, you will need to install GNU-Ada. That is fairly simple on Linux systems, and somewhat more complex on OSX. One good source is:

	https://github.com/alire-project/GNAT-FSF-builds/releases

For Windows users, one needs to build a 64-bit executable to access all available memory.
Please read the details in the file "gnuAdaOnWindows.txt".


## Shameless Plug for my own Sokoban game-platforms:

This solver, and 2 others, is embedded "live" in my three games (for Windows,Osx,&Linux):

	* RufaSok (plain, with several skins)
	* WorldCupSokerban (soccer-themed that uses "balls" shaped like the intersection of 2 cylinders)
	* SliderPuzzles (collection of ascii, non-graphical, puzzles including sokoban)

To me "live" means that the solver can be invoked at any time and it attempts to solve, not the original state, but the current state of your sokoban puzzle (whether or not it is still solvable). Using a keyboard key, it single steps toward a solution, but can be de-invoked at any time after it has gotten you out of a difficult situation, and you think you can complete the solution by yourself. That capability is invaluable to helping one to learn to manually solve sokoban puzzles.

The embedded solvers are time limited. You can set the timeout, but the default is 10sec.

See:

https://sourceforge.net/projects/rufassok/

https://sourceforge.net/projects/worldcupsokerban/

https://sourceforge.net/projects/sliderpuzzles/


--------------------------
## License:


This app is covered by the GNU GPL v3 as indicated in the sources:


Copyright (C) 2024  <fastrgv@gmail.com>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You may read the full text of the GNU General Public License
at <http://www.gnu.org/licenses/>.

---------------------------------------------
keywords:
hungarian, ada, munkres, kuhn, kuhn-munkres,
puzzle, sokoban, solver

===================== update history ========================

**ver 1.1.3 -- 21nov2023**
* Revised an internal list structure; changed a LIFO stack into a FIFO queue. This means that among equal-priority configurations, the first one found is processed first. This is a more typical design, but new to hbox5. The push/move efficiency of solutions are somewhat improved (but not speed).

**ver 1.1.2 -- 16nov2023**
* Added an option to disable 3 of 4 priority measures (to enable a "baseline" method).

**ver 1.1.1 -- 15nov2023**
* Added a 5th & 6th solution methods, for the sake of completeness. 
* Now include consideration of puller-to-pullerGoal accessibility.
* Improved method for determining "endgame", the point at which 3 measurands are dropped.
* Please read "~/docs/changes15nov23.txt" for details.

**ver 1.1.0 -- 10nov2023**
* Corrected the recalculation of priority when reaching the same configuration with fewer pulls (Hungarian methods only). This fix allowed the default method (#0) to solve 3 more puzzles from Xsokoban-90.
* Corrected the non-hungarian method (#2) to [properly] use a simple boxes-on-goals count. Note that this change typcally reduced the speed & effectiveness of method 2. Recall that this method exists only to show the true advantage of using the Hungarian algorithm.
* Added a 4th method that considers total moves, to produce solutions with less dithering. This method should be the new default but it's not, in order to be backwards compatible. But it is fast and removes the very embarrasing dithering one sees in [the default] method #0. It also solves 4 more puzzles from Xsokoban-90, bringing the total to 40.

**ver 1.0.7 -- 31oct23**
* Revised OSX build; does not use any Xcode.

**ver 1.0.6 -- 08nov2022**
* Windows 64-bit build now uses simple-to-install GNU Ada compiler.
* Enhanced algorithmic explanations.

**ver 1.0.5 -- 22sep2022**
* Moved source code into ./src/.
* Moved build scripts into ./build/.
* Converted to use 64-bit GNU Ada rather than defunct AdaCore compiler.

**ver 1.0.4 -- 3mar2021**
* Simplified, yet expanded the commandline parameters. Now allow naming an output file.
* A linux executable [hbox5] is also included.

**ver 1.0.3 -- 18feb2021**
* hbox5 now aborts when solution is not found after 10 minutes, to facilitate batch runs.

**ver 1.0.2 -- 12feb2021**
* Code cleanup;
* Replaced splaylist with simpler splaytree;
* Included both 32-bit and 64-bit Windows executables;

**ver 1.0.1 -- 10feb2021**
* added a 3rd solution method option to omit the Hungarian estimator.
* now deliver a Windows executable [runs fine under Wine]

**ver 1.0.0 -- 08feb2021**
* Initial release


