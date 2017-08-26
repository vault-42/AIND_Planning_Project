# Planning Search Project Implementation 
### Udacity Artificial Intelligence Nanodegree Program
## Synopsis
The objective of this project is to implement planning AI agents for fully observable, deterministic, static environments with single agents. In particular I developed planning agents for the air cargo transport “world” discussed in Artificial Intelligence: A Modern Approach (AIMA) section 10.1.1 in the 3rd edition <sup>[1](#myfootnote1)</sup>. To represent the world the project uses the Planning Domain Definition Language first discussed in section 10.1. Then I run uninformed planning searches (breadth first search, depth first graph search, etc) and use them as a baseline to compare their performance against the A* planning graph search using domain-independent heuristics. This provides a good comparison between "uninformed" planning and "informed" planning (distinctions discussed below).

## Code
This project requires Python 3. Project used a pre-packaged Anaconda3 environment provided by the Udacity staff for the artificial intelligence nanodegree program.

* `my_air_cargo_problems.py` - Contains the code to solve the logisitics planning problems.
* `my_planning_graph.py` - Contains the code to implement the planning graph.
* `tests/test_my_air_cargo_problems.py` - Provided by Udacity staff to test the implementation.
* `tests/test_my_planning_graph.py` - Provided by Udacity staff to test the implementation.
* `run_search.py` - Provided by Udacity staff to evaluate the performance of the implementation.

## Problem Definition
The project problem is set in the air cargo transport "world" from the Artificial Intelligence: A Modern Approach section 10.1.1 in the 3rd edition <sup>[1](#myfootnote1)</sup>. In this "world" all problems can use the following action schema:

![Alt text](/images/PDDL_AirCargo_ActionSchema.png?raw=true)

*PDDL description of air cargo transportation action schema*

Three problems were given all with a different initial state and goal:

![Alt text](/images/problem1.png?raw=true)

*Problem 1*

![Alt text](/images/problem2.png?raw=true)

*Problem 2*

![Alt text](/images/problem3.png?raw=true)

*Problem 3*

### Optimal Flight Plans
All of these problems have optimal flight plans (shortest number of actions to reach goal). Provided are example optimal flight plans:

| **Problem 1**       | **Problem 2**       | **Problem 3**       |
| ------------------- | ------------------- | ------------------- |
| Load(C1, P1, SFO)   | Load(C1, P1, SFO)   | Load(C2, P2, JFK)   |
| Load(C2, P2, JFK)   | Load(C2, P2, JFK)   | Fly(P2, JFK, ORD)   |
| Fly(P1, SFO, JFK)   | Load(C3, P3, ATL)   | Load(C4, P2, ORD)   |
| Fly(P2, JFK, SFO)   | Fly(P1, SFO, JFK)   | Fly(P2, ORD, SFO)   |
| Unload(C1, P1, JFK) | Fly(P2, JFK, SFO)   | Load(C1, P1, SFO)   |
| Unload(C2, P2, SFO) | Fly(P3, ATL, SFO)   | Fly(P1, SFO, ATL)   |
|**Optimal Length: 6**| Unload(C3, P3, SFO) | Load(C3, P1, ATL)   |
|                     | Unload(C2, P2, SFO) | Fly(P1, ATL, JFK)   |
|                     |**Optimal Length: 9**| Unload(C3, P1, JFK) |
|                     |                     | Unload(C2, P2, SFO) |
|                     |                     | Unload(C1, P1, JFK) |
|                     |                     |**Optimal Length: 12**|

### Code Execution
To run the program with the same search algorithms described below execute the following command at the command line:
`python run_search -p 1 2 3 -s 1 3 5 7 8 9 10`
or run the following command to go into interactive mode:
`python run_search -m`

## Uniformed Planning Search Results and Analysis
These searches are considered “uninformed” due to the fact that they aren’t extracting extra information from the states. These searches are simply expanding nodes according to their heuristic until they find a goal state. For each problem I compare the relative performance of the uniformed planning searches over the following criteria: execution time, search node expansions (correlates to memory usage), path length, and whether or not the solution is optimal. *All execution times below are in seconds*.

| **Search Type**         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| ----------------------- | ------------------:| ------------------:| --------------:| -----------:|
| breadth_first_search    |                .053|                  43|               6|          Yes|
| depth_first_graph_search|                .015|                  21|              20|           No|
| uniform_cost_search     |                .038|                  55|               6|          Yes|
| greedy_best_first_search|                .005|                   7|               6|          Yes|

*Uninformed Problem 1 Results*

| **Search Type**         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| ----------------------- | ------------------:| ------------------:| --------------:| -----------:|
| breadth_first_search    |              13.176|                3346|               9|          Yes|
| depth_first_graph_search|                .309|                 107|             105|        No!!!|
| uniform_cost_search     |               11.15|                4852|               9|          Yes|
| greedy_best_first_search|               2.365|                 990|              15|           No|

*Uninformed Problem 2 Results*

| **Search Type**         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| ----------------------- | ------------------:| ------------------:| --------------:| -----------:|
| breadth_first_search    |              92.366|               14120|              12|          Yes|
| depth_first_graph_search|               1.062|                 292|             288|        No!!!|
| uniform_cost_search     |              48.923|               18235|              12|          Yes|
| greedy_best_first_search|              15.112|                5614|              22|           No|

*Uninformed Problem 3 Results*


Breadth_first_search and uniform_cost_search are the only two optimal searches. Between the two, uniform_cost_search is slightly faster but uses slightly more memory (i.e. more node expansions). Depth_first_graph_search is (comparatively) blazing fast and uses less memory but comes up with a **very** non-optimal plan. Greedy_best_first_search is (compared to the other searches) more “well-rounded”. The execution time and node expansions can be much less than breadth_first_search
and uniform_cost_search. The path length, however, was not always optimal but much shorter than depth_first_graph_search.

Given that most modern computers have plenty of memory and that finding the optimal path length is often of utmost importance, uniform_cost_search is likely the best search heuristic. If finding the optimal path is not critical, then greedy_best_first_search would be a solid pick.

## A* Planning Graph Search Results and Analysis
These searches are considered "informed" due to their extraction of extra information from the problem definition. The A* search uses an "admissible" heuristic by defining a relaxed problem from the original problem and estimating the associated cost. The heuristic then uses this cost estimate to more intelligently determine the search direction. I'll look at the same result criteria as defined in the previous section.

| **Search Type**                         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| --------------------------------------- | ------------------:| ------------------:| --------------:| -----------:|
| A* search with h1 heuristic             |                .060|                  55|               6|          Yes|
| A* search with ignore precond heuristic |                .049|                  41|               6|          Yes|
| A* search with level sum heuristic      |               1.009|                  11|               6|          Yes|

*A\* Problem 1 Results*

| **Search Type**                         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| --------------------------------------- | ------------------:| ------------------:| --------------:| -----------:|
| A* search with h1 heuristic             |              11.193|                4852|               9|          Yes|
| A* search with ignore precond heuristic |               4.156|                1450|               9|          Yes|
| A* search with level sum heuristic      |             193.543|                  86|               9|          Yes|

*A\* Problem 2 Results*

| **Search Type**                         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| --------------------------------------- | ------------------:| ------------------:| --------------:| -----------:|
| A* search with h1 heuristic             |              48.814|               18235|              12|          Yes|
| A* search with ignore precond heuristic |              16.099|                5040|              12|          Yes|
| A* search with level sum heuristic      |             796.637|                 325|              12|          Yes|

*A\* Problem 3 Results*

All A* search heuristics solve all 3 problems optimally. Using the ignore precond heuristic resulted in the fastest times and was even more memory efficient than the second fastest heuristic (h1). The level sum heuristic should only be used when the computer is memory constrained. Level sum uses far less memory (lower node expansion count) but is **much slower** than all other heuristics. 

## Conclusion
Looking at the table below, a comparison of the fastest uninformed search vs the fastest informed search, clearly shows the benefits of extracting extra information from the original problem. The A* search with ignore preconditions heuristic outperforms in speed and memory across all three problems save in speed for problem 1. Given that problem 1 requires the fewest amount of levels to solve, informed searches aren't worth the effort. For Problem 3, not only is it ~3x faster but it is also ~4x more memory eficient. A* search with the ignore preconditions heuristic is the best search heuristic for the air cargo transportation planning problem.

| **Search Type**                         | **Execution Times**| **Node Expansions**| **Path Length**| **Optimal?**|
| --------------------------------------- | ------------------:| ------------------:| --------------:| -----------:|
| **Problem 1**                                                                                                    |
| uniform_cost_search                     |                .038|                  55|               6|          Yes|
| A* search with ignore precond heuristic |                .049|                  41|               6|          Yes|
| **Problem 2**                                                                                                    |
| uniform_cost_search                     |               11.15|                4852|               9|          Yes|
| A* search with ignore precond heuristic |               4.156|                1450|               9|          Yes|
| **Problem 3**                                                                                                    |
| uniform_cost_search                     |              48.923|               18235|              12|          Yes|
| A* search with ignore precond heuristic |              16.099|                5040|              12|          Yes|

*Uniform Cost Search vs A\* search with ignore preconditions direct comparison*

## Bonus: PyPy3 vs CPython Performance Comparison
In this section I'll take a look at the performance of PyPy3<sup>[2](#myfootnote2)</sup>, a JIT interpreter known for excellent execution speed, versus CPython, the standard reference Python interpreter on this project. Since this project has no compatibility issues with the PyPy3 interpreter it is a good candidate for a performance comparison.

Since I already have the execution times running on the CPython interpreter, all I need to do is run the same command with the PyPy3 interpreter as follows:
`pypy3 run_search.py -p 1 2 3 -s 1 3 5 7 8 9 10`

The table below shows the execution times for each Interpreter and the speedup relative to the CPython Interpreter.

| **Search Type**                         | **CPython Execution Times (s)**| **PyPy3 Execution Times (s)**| **Speedup**|
| --------------------------------------- | ------------------------------:| ----------------------------:| ----------:|
| **Problem 1**                                                                                                        |
| breadth_first_search                    |                            .053|                          .091|        .55X|
| depth_first_graph_search                |                            .015|                          .029|        .52X|
| uniform_cost_search                     |                            .038|                          .064|        .59X|
| greedy_best_first_graph_search          |                            .005|                          .008|        .63X|
| A* search with h1 heuristic             |                            .060|                          .052|       1.15X|
| A* search with ignore precond heuristic |                            .049|                          .047|       1.04X|
| A* search with level sum heuristic      |                           1.009|                          .552|       1.83X|
| **Problem 2**                                                                                                        |
| breadth_first_search                    |                          13.176|                         3.687|       3.57X|
| depth_first_graph_search                |                            .309|                          .083|       4.70X|
| uniform_cost_search                     |                           11.15|                         4.217|       2.64X|
| greedy_best_first_graph_search          |                           2.365|                          .883|       2.68X|
| A* search with h1 heuristic             |                          11.193|                         4.170|       2.68X|
| A* search with ignore precond heuristic |                           4.156|                         2.483|       1.67X|
| A* search with level sum heuristic      |                         193.543|                         8.982|      21.55X|
| **Problem 3**                                                                                                        |
| breadth_first_search                    |                          92.366|                        16.730|       5.52X|
| depth_first_graph_search                |                           1.062|                          .251|       4.23X|
| uniform_cost_search                     |                          48.923|                        15.718|       3.11X|
| greedy_best_first_graph_search          |                          15.112|                         4.830|       3.13X|
| A* search with h1 heuristic             |                          48.814|                        15.733|       3.10X|
| A* search with ignore precond heuristic |                          16.099|                         8.180|       1.97X|
| A* search with level sum heuristic      |                         796.637|                        42.776|      18.62X|

The results show that while PyPy3 doesn't always result in speed up, in most cases PyPy3 will significantly improve the execution time. This is especially true for the algorithms and problems that took greater than 5 seconds to run with the CPython interpreter. Once PyPy is installed and if the program is already [compatible with PyPy](https://pypy.org/compat.html), it is really easy to run PyPy. With this in mind, my take away is to try PyPy for every program that is supported by PyPy and is performance sensitive.

## Research Review
For this project submission we also needed to submit a page and a half review related to research in the field of AI planning. I chose to write a research review on the advancements in the *International Planning Competition (IPC)* <sup>[3](#myfootnote3)</sup> held every year. Please see my `research_review.pdf` file to read the review.

## References
<a name="myfootnote1">[1]</a>  *Artificial Intelligence*, 3rd Edition, by Russelll, Stuart, Norvig, Peter, published by Pearson Education, Limited. Copyright 2010 by Pearson Education, Inc.

<a name="myfootnote2">[2]</a> "What is PyPy?", [https://pypy.org/features.html](https://pypy.org/features.html)

<a name="myfootnote3">[3]</a> "ICAPS Competitions", [www.icaps-conference.org/index.php/Main/Competitions](www.icaps-conference.org/index.php/Main/Competitions)
