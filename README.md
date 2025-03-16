![screenshort](https://github.com/fastrgv/hbox4/blob/main/t7s.png)

Here is a link to all source code and build files:

https://github.com/fastrgv/hbox/releases/download/v1.3.2/hb3mar25.7z

Type "7z x filename.7z" to extract the archive.

* On OSX, Keka works well for 7Z files. The command-line for Keka is:
	* /Applications/Keka.app/Contents/MacOS/Keka --cli 7z x (filename.7z)





#### Note to github downloaders: (Please ignore the "Source code" zip & tar.gz files, which are auto-generated by GitHub). Download the latest release by clicking on the large 7z file under releases. It contains all assets for Windows & Linux, including source code. Type "7z x filename" to extract the archive. Then you can compile your own binaries, or use the pre-built ones provided.














https://sourceforge.net/projects/hbox4/
	or
https://sourceforge.net/projects/hbox4/files/latest/download

# hbox -- sokoban solver using Ada



#### What's new:


**ver 1.3.3 -- 17mar2025**

* Now request & verify real-time priority for Windows [necessary in W11 according to my tests].
* Added a minimal puller-deadlock guard.
* Refined the threshold function that determines "halfway".
* New default method is now "move-efficient" meth#1.


**ver 1.3.2 -- 3mar2025**

* Improved coding & data structures. Six splaytrees now replace hundreds of hashlists.
* Improved indexing now allows more boxes & larger puzzles.
* Added binaries for users of old linux under ~/oldLinux/.
* When the commandline includes an output filename, screen output is now suppressed.


**ver 1.3.1 -- 11feb2025**

* Corrected a logic error affecting revisited box configurations.
* Added "inertia", meaning pulls are repeated if advantageous.
* Removed old solution methods 3,4,&5 for enhanced simplicity.
* Replaced old method 3 with a single step method, equivalent to the previous version of hbox.
* Added new method 4 that omits the 6th heuristic, that is rarely useful.
* Added an additional sanity check on memory and aborts when when available memory is very low.


**ver 1.3.0 -- 25jan2025**

* Simplified the name to hbox.
* Added 6th heuristic that drives searching for alternate configurations with equal promise.
* Added better description of "baseline" solution methods 10..15, which are often quite usable, but less robust.
* Renamed the terminology "endgame" to "halfway" since it's more descriptive. This signifies a point in the solution search where several heuristics are dropped because their utility has ended.


#### More change-history at end of this file



## Description

Hbox is a commandline-terminal sokoban solver written in Ada; a multiple step box search that uses the Hungarian Algorithm, and a reverse solution technique.

It is "generic" in the sense that it contains no domain specific strategies. 

-----------------------------------------------------------
Featuring

	* no installation
	* no dependencies
	* simply unzip in your Downloads directory, and run.
-----------------------------------------------------------

Pre-built executables are provided in 3 variants:

	* hbox.exe (Win64, including Windows 10 & 11)
	* hbox_gnu (linux)
	* hbox_osx (Mac/OSX)


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

Sokoban puzzles typically come in files containing groups of puzzles, so the user interface assumes that you provide a file-name, and 1 integer representing the number of the puzzle to be solved. The solver name is "hbox".

EG: hbox games/Sladkey.sok 22

is the command to solve level 22 from the file "Sladkey.sok". The solution is written as a long string in the terminal window at the end of processing. Of course it could be redirected to a file thusly:

EG: hbox games/Sladkey.sok 22 > soln.txt

-------------------------------------------------------------
In addition to the 2 mandatory commandline parameters discussed above, there are 4 more optional ones:

* (3) [float] MaxGb memory to use
* (4) [int 0..4, 10..14] Solution method:
	* 0 (default) Quick solution using "smart-inertia" [a cfg updated only if #pushes is reduced]
	* 1 Move-reducing updates using inertia [a cfg updated even if #pushes is equal but #moves is reduced]
	* 2 No Hungarian Estimator using inertia: possibly more move-efficient solutions but typically slower.

	* 3 Method 0 without inertia, i.e. single-steps. (similar to hbox6, the prior version)

	* 4 Suppresses use of the sixth heuristic. (similar to hbox5, an older version)

	* 10..14 triggers "baseline" option for the above 5 methods where only one or two heuristics are used. So simply add 10 to the method number 0..4 to get its "baseline" version. 

		The methods 10,11,13 use only 2 heuristics (#1,#6), while 12,14 only one (#1). The 6 Heuristics (priority measures) are explained below.


* (5) [integer] TimeoutSec
* (6) OutputFileName

EG: hbox games/Sladkey.sok 22 6.5 1 600 sladkey22.txt

indicates using no more than 6.5 Gb of memory and the efficient! solution method, a timeout of 600 seconds, AND to write the output to a file in the current directory named "sladkey22.txt". !But note that the solutions are not always efficient. This name merely refers to certain algorithmic choices within the solver which generally tend to produce solutions with fewer moves.

EG: hbox games/Sladkey.sok 22

indicates using the defaults, i.e. 6.5Gb memory and fastest solution.

EG: hbox games/Sladkey.sok 22 5.5 10

indicates method 0 but using "baseline" single priority measure for comparison purposes.

-------------------------------------------------------------------------------

There are many puzzles this algorithm will not solve due to time or memory limits, so the embedded memory limiter will exit gracefully when memory usage exceeds the preset limit, OR if actual available memory has been reduced to less than 5% of the original amount (by this app. or by some other concurrent app).

Finally, if you don't want to wait for the solver to finish, you can (ctrl)-c out of it to quit immediately.



## Algorithm Used

A multiple-heuristic, multiple-step box search, done in **reverse**, using six "orthogonal" priority measures [heuristics] in a round-robin sequence. The Hungarian Algorithm is used to match boxes with goals and dynamic programming generates an estimate of the number of future box moves to solution, called "HunEst".

An article by Frank Takes shows advantages to working from a solved position backwards to the start position. This prevents box-deadlocks from taking up space in the search tree. Thusly, the formidable issue of deadlock avoidance is completely ignored. Likewise, the tricky issue of goal-packing-order is sidestepped, as well.

A self balancing splaytree is used to test whether a given configuration was seen before. There are also 6 priority queues embedded into the splaytree that provide distinct "views" of the data.

The first 4 priority measures are pretty straight forward.  They were adapted from the "Festival" algorithm description, to allow rapid evaluations:

* pri1: Nboxes - NboxesOnGoals.........0 means all boxes on goals
* pri2: Ncorrals - 1				.........0 means only 1 corral, i.e. pusher is free to roam
* pri3: NblockedRooms			.........0 means no boxes block room-doors
* pri4: NblockedBoxes			.........0 means all boxes are pushable

* pri5: Furthest-Boxes First  .........non-linearly coerces most-distant boxes nearer their goals

* pri6: Exploratory				.........drives exploration of alternate, promising configurations

Note that priority measure #1 counts a box as being on a goal only if it lies on its Hungarian-assigned goal, except when non-Hungarian solution methods are used.

Note also that pri5 and pri6 are not available for nonHungarian methods #2 and #5.

All 6 priority measures are small non-negative integers (1..60) to be minimized. So it often occurs that there are many candidate nodes of the search-tree with the same measure. 

So to help distinguish those that are more promising, a secondary priority measure [pri0] is used that counts the total box-pulls to reach the current configuration, **PLUS** the Hungarian-Algorithm-based estimate (HunEst) of the remaining number of pulls. I.E.

* pri0 := #pulls + HunEst

which currently has a range limit of 0..700.


### Inertia

Inertia refers to taking more than one box-pull in each direction when it lands on a grid-cell with little value, or in a tunnel. Intermediate steps are all saved, however. The net effect is a slight gain in efficiency.



### The 5th heuristic

The 5th priority measure is described as follows:

	* Nfar = 60(bmx0-bmx)/bmx0

(60 is the maximum numerical value allowed for priority measures 1 thru 5)

where bmx is the sum of the squares of the box distances to their hungarian-matching goal.

The desired behavior, in the forward game, is to clear out the boxes closest to their targets first, so subsequent moves have fewer obstacles. 

Thus, in the reverse game, we want to motivate placing the boxes furthest from their goal **before** we address the boxes relatively close to their goal.
Yes, there are already heuristics that drive boxes toward their goals, but they are linear in their effect. The intent here is to create a nonlinear effect that would prioritize the more distant traversals. My motivating example was problem #4 from the classic set of 90.




### The 6th heuristic

The 6th priority measure, Nalt, works differently, to promote meandering. It is defined as follows:

	* Nalt = 60(HunEst)/HunEst0
	* persists beyond "halfway"

where HunEst0 is the hungarian estimated number of moves to solution at the initial puzzle configuration. Over time this estimate generally decreases to zero. HunEst is the current estimate.

Pri6 is a reasonable "neutral" estimator of merit that does not penalize moves & pushes, thus encouraging safe explorations. Pri2, pri3, & pri4 might stifle proper endgame maneuvers that require closing doors to rooms, and access to boxes on goals, so are not neutral estimators of merit when used late in the solution.

Safe exploration is the embodiment of the general puzzle-solving principle of simple rearrangement to find an alternate configuration that is easier to solve. Of course this strategy is not specific to sokoban.

My motivating example here is puzzle 11 of the original 90, where a single blocking box needs to be moved to a non-blocking position before the puzzle is then easily solved. The previous version of hbox could NOT solve this one. Subsequently, I found that several other puzzles are either newly solvable, or more quickly solved.



### Six working together

Finally, the round robin regimen that initially includes 6 measures, and eventually drops 2,3,4, &5, somewhere beyond the halfway point since corrals, blocked rooms & boxes must eventtually be permitted at the end of the reverse game, i.e. near the beginning of a forward game.

So after the "halfway" only 2 measures still operate: pri1, pri6. I found that the 6th measure was still needed after the halfway point in the solution process.


### Criteria for "HalfWay"

BoG, a recent average #boxes on goals, is tracked and halfway is declared when BoG > 2/3 #boxes.

The priorities 2, 3, 4 & 5 eventually become counter-productive, and are dropped. 

------------------------------------------------------------------------------

### Additional algorithmic details

A nodal configuration description consists of the box layout, the upperleft corner of the puller-corral, as well as the total box pulls, the hungarian estimate of the future box pulls, and the 6 priority measures.

One extra condition required of the reverse solution is that a path exists for the puller to reach its goal, i.e. the starting pusher position.

Conceptually, the Splaytree Priority Queue {frontier} contains 6 independent queues, each ordered by one of the 6 priority measures mentioned above. So if the round-robin parameter is K in {0,1,2,3,4,5}, we greedily pop the leading candidate node off of the front of queue #K, thereby deleting it from {frontier}.

So, in this MultiStepBoxSearch the initial configuration is read, evaluated and pushed into the {frontier} queue. But goal and boxes are interchanged because this is a backward search using a "puller" rather than a "pusher".

So here begins the description of the main loop:-------------------------------------------

Choose "K", and pop the frontrunner off Kth queue and look at its nodal information to retrieve the hashkey. This key allows direct access via the splaytree structure that, in turn allows direct removal of the node from the splaytree and all six PriQueues. Thusly, the removals from the other PriQueues do not require any time consuming list-traversals.

This removed configuration is then tested to see whether a solution has been reached. If not, it is inserted into the {explored} splaytree, which has no queue structures attached. The splaytree allows rapid insertion and rapid random retrieval in case we find a solution and need to reconstruct our path. 

For this current configuration we simply cycle through each [unsorted] box and try to move it one or more steps in each direction. If a move is possible, each new box configuration is evaluated and enqueued into the {frontier} set. This involves a splaytree insertion and 6 priority-queue insertions that each maintain a separate ordering.

End of main loop.------------------------------------------------------

Hbox is now a multi-step algorithm that tries to repeat steps when possible.
If the box-density is high, then single-step mode is recommended. When running single-step, this algorithm is identical to older versions of hbox. By the way, if the box-density is high, it is likely that a non-Hungarian method is preferrable, i.e. methods 2 or 12. High density means a compact intermixing of boxes and goals.

### Minimal Puller-Deadlock Avoidance Pattern

Working backward avoids many problems, but I noticed that impossible intermediate states can still occur. When making a move, I now require that a box be pullable from at least one direction, unless it is on a goal.

### SplayTree-Priority-Queue

The splaytree [self-balancing-binary-tree] based priority queues allow unique hash keys to be inserted, found & removed quickly, with 6 embedded priority queues that allows efficient insertions, access, and deletion, both from the heads [popping], and directly by key. The hash keys uniquely identify pusher & box-layouts at each saved node, to avoid duplicates. The 6 priority queue orderings allow primary and secondary (tie-breaker) priority measures. As mentioned above, the structure also allows rapid, direct access deletions given the hashkey [O(log n)].

It was very difficult to get these priority-queue operations to be efficient enough for use in a sokoban solver. The speed is really quite remarkable!

For further insights about the functional details of the frontier data set handling see:  "~/docs/frontier.txt" and "~/docs/doubleSplay.txt"


### Dynamic Programming [flood-fill]
Dynamic programming allows efficient determination of box-valid locations and the feasibility and minimal cost of traversing between two locations. This information is used to feed into the Hungarian Algorithm.

### Hungarian Matching Algorithm
This fully functional implementation of the Hungarian algorithm provides valid, one-to-one pairings between boxes and goals for each intermediate box-layout, and provides realistic estimates [admissible & consistent] of the number of box-moves remaining to completion. The power of this heuristic can be seen when it is swapped out; i.e. when the secondary priority measure is changed from (moves + estimated-future-moves) to just (moves), the solver is still functional, and even faster on a some puzzles, but is significantly impaired on many others.

* admissible means: heuristic(x,goal) <= actualCost(x,goal)
* consistent[monotone] means: h(x,g) <= actualCost(x,y)+h(y,g)

Note that consistent => admissible assuming h(g,g)=0.

One may choose to omit the Hungarian estimator so that the secondary [tie-breaking] priority measure then becomes simply the total box moves used to arrive at a given configuration.

#### Note on the Hungarian Algorithm: 
I have searched for a correct version online, but found none. I found several that "almost" worked but were all flawed, mainly, I think, due to the age and nature of the original algorithmic description. It was invented before computers were widely available, so was described in terms of hand computations, parts of which are quite confusing, possibly due to language ambiguities. [Kuhn, 1955]

The algorithm used here was copied on 20sep18 from: https://users.cs.duke.edu/~brd/Teaching/Bio/asmb/current/Handouts/munkres.html [now, a dead link] and modified to correct some errors. It has worked flawlessly for several years, now.




## What's so great about this app?

By today's standards, this is only a moderately capable sokoban solver, solving about 57 of the original 90 (RollingStone solved 59). What makes it so interesting and unique is its simplicity and utter ignorance! It is unlikely that you will find another sokoban solver that knows LESS about the game of sokoban.

These qualities result from a minimalistic regimen that AVOIDS:

* complex control mechanisms;
* domain-specific strategies or tactics;
* databases;
* macro-moves, tunnel-macros, goal-packing-macros;
* matching of box-pattern-templates;

This Ada code is currently written in a relatively abstract, algorithmic style, to enhance the transparency of intent. But this also means that there are potentially considerable savings to be made in compactness and speed by using low, bit-level operations instead.

OTOH, for the experienced Ada programmer there are many improvements possible to better utilize the advanced protections and features of the Ada language.

### Other Valuable Attributes

* easily buildable on Windows, OSX, and Linux using a free GNU Ada compiler.
* no dependencies; no installation.
* uses algorithms and data structures of general interest and usefullness.
	* Hungarian Algorithm to help direct effort.
	* Splaytrees for accessing data.

Hbox adds further proof of the effectiveness of the design choices made in the Festival solver that proposed the original 4 "orthogonal" heuristic measures, or so-called "features".

Note that the splaytree priority queue and the Hungarian Algorithm represent valuable, stand-alone, Ada packages, or C++ classes, that would be worth extracting for other uses.


## Shortcomings

Disclaimer #1: the elegance lies in the algorithms and data structures, not the code.

Disclaimer #2: No attempt at solution optimality is made. The goal in this solver is to find any solution. The 6 orthogonal "features" do not lend themselves to finding solutions with any type of optimality. Even using methods 1 or 3 for a more efficiency, the resulting solutions can sometimes appear pretty stupid, with strange moves produced that are clearly wasteful. Still, the solver is surprising in its capability, considering its lack of domain-specific knowledge, which was a deliberate design choice.

In any case, I wish to expose this algorithm to public scrutiny, and allow anyone with an interest, the chance to improve or extend this generic approach to the formidable task of solving sokoban puzzles.


## Xsokoban Levels Solved (updated early 2025):

Hbox with inertia solves 51 of 90 puzzles by method 0, and 6 more by other methods.

See ~/docs/runtimes_Inertia.txt

All failures I have seen are due to shortage of memory or time.


## Build Instructions:

Three usable, pre-compiled binary executables are delivered (Windows/Linux/OSX). But if you choose to try to rebuild, you will need to install GNU-Ada. That is fairly simple on Linux systems, and somewhat more complex on OSX. One good source is:

	https://github.com/alire-project/GNAT-FSF-builds/releases

For Windows users, one needs to build a 64-bit executable to access all available memory.
Please read the details in the file "gnuAdaOnWindows.txt".


## See also my Sokoban game-platforms:

This solver, and 2 others, is embedded "live" in my three games (for Windows,Osx,&Linux):

	* RufaSok (minimalistic, with several skins)
	* WorldCupSokerban (soccer-themed that uses "balls" shaped like the intersection of 2 cylinders)
	* SliderPuzzles (collection of ASCII, non-graphical, puzzles including sokoban)

To me "live" means that the solver can be invoked at any time and it attempts to solve, not the original state, but the current state of your sokoban puzzle (whether or not it is still solvable). Using a keyboard key, it single steps toward a solution, but can be de-invoked at any time after it has gotten you out of a difficult situation, and you think you can complete the solution by yourself. That capability is invaluable to helping one to learn to manually solve sokoban puzzles.

The embedded solvers are time limited. You can set the timeout, but the default is 10sec. Obviously, this time limit reduces the capabilities, versus the command line solver.

See:

https://sourceforge.net/projects/rufassok/

https://sourceforge.net/projects/worldcupsokerban/

https://sourceforge.net/projects/sliderpuzzles/


--------------------------
## License:


This app is covered by the GNU GPL v3 as indicated in the sources:


Copyright (C) 2025  <fastrgv@gmail.com>

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

**ver 1.2.0 -- 27dec2024**
* Added back a 6th command line parm: outputFileName (for sokoban YASC).
* Fixed a problem with the embedded version of hbox5, method 3.

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


