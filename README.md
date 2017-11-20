
This project solves deterministic logistics planning problems
for an Air Cargo transportation. We first construct a compact data
structure called planning graph, then compare the performance
of Forward state-space search algorithms with different heuristics.

This project was submitted as part of the completion for
 [Udacity’s Artificial Intelligence Nanodegree program](https://github.com/udacity/AIND-Planning)

# Three air cargo transportation problems


- Air Cargo Action Schema:
```
Action(Load(c, p, a),
	PRECOND: At(c, a) ∧ At(p, a) ∧ Cargo(c) ∧ Plane(p) ∧ Airport(a)
	EFFECT: ¬ At(c, a) ∧ In(c, p))
Action(Unload(c, p, a),
	PRECOND: In(c, p) ∧ At(p, a) ∧ Cargo(c) ∧ Plane(p) ∧ Airport(a)
	EFFECT: At(c, a) ∧ ¬ In(c, p))
Action(Fly(p, from, to),
	PRECOND: At(p, from) ∧ Plane(p) ∧ Airport(from) ∧ Airport(to)
	EFFECT: ¬ At(p, from) ∧ At(p, to))
```

- Problem 1 initial state and goal:
```
Init(At(C1, SFO) ∧ At(C2, JFK)
	∧ At(P1, SFO) ∧ At(P2, JFK)
	∧ Cargo(C1) ∧ Cargo(C2)
	∧ Plane(P1) ∧ Plane(P2)
	∧ Airport(JFK) ∧ Airport(SFO))
Goal(At(C1, JFK) ∧ At(C2, SFO))
```
- Problem 2 initial state and goal:
```
Init(At(C1, SFO) ∧ At(C2, JFK) ∧ At(C3, ATL)
	∧ At(P1, SFO) ∧ At(P2, JFK) ∧ At(P3, ATL)
	∧ Cargo(C1) ∧ Cargo(C2) ∧ Cargo(C3)
	∧ Plane(P1) ∧ Plane(P2) ∧ Plane(P3)
	∧ Airport(JFK) ∧ Airport(SFO) ∧ Airport(ATL))
Goal(At(C1, JFK) ∧ At(C2, SFO) ∧ At(C3, SFO))
```
- Problem 3 initial state and goal:
```
Init(At(C1, SFO) ∧ At(C2, JFK) ∧ At(C3, ATL) ∧ At(C4, ORD)
	∧ At(P1, SFO) ∧ At(P2, JFK)
	∧ Cargo(C1) ∧ Cargo(C2) ∧ Cargo(C3) ∧ Cargo(C4)
	∧ Plane(P1) ∧ Plane(P2)
	∧ Airport(JFK) ∧ Airport(SFO) ∧ Airport(ATL) ∧ Airport(ORD))
Goal(At(C1, JFK) ∧ At(C3, JFK) ∧ At(C2, SFO) ∧ At(C4, SFO))
```

# Planning with state-space search with heuristics

1) Ignore preconditions heuristics

Let’s first consider the approach adding edges to the graph.
To relax the problem, we ignore all preconditions for each action
 and all literals except those in the goals.
 With these assumptions, every action is executable in all states
 and any single goal can be attained by a single step.
 This heuristic computes the minimum number of steps required to
 so that the union of effects of actions can achieve the goals.
 For our case, it’s equivalent to counting the number of
 unsatisfied goals.

2) level sum heuristics

Once constructed a planning graph, we can estimate the cost
of achieving any goal literals
$g_i$ as the level a which
$g_i$ first appears in the planning graph from the initial state.
 This is called the level sum.

We implement the level sum heuristic, sum of the level costs of
 the goals. This heuristic is inadmissible, but works well in
 practice.


# Search result
### Problem 1

To achieve the goal of the first problem, it is required
to move the cargo 1 from SFO to JFK, and cargo 2 from JFK to SFO.
The optimal solution has a sequence of six actions that loading each cargo
to the planes at each airport (2), flying both planes to the other airports (2)
and unloading the cargo (2).
An example of optimal solution is
\[Load(C2, P2, JFK), Load(C1, P1, SFO), Fly(P2, JFK, SFO),
Unload(C2, P2, SFO), Fly(P1, SFO, JFK), Unload(C1, P1, JFK)].

##### Comparison of uninformed search for three problems
Maybe because this problem is relatively small, uninformed search algorithms
could find the optimal planning solution in fairly short time.
 Though the depth first graph search algorithm and depth limited search
 algorithm explored smaller number of new nodes and used less data
 storage in shorter runtime, the solution they found were not optimal,
 including redundant actions. This is because the search is not complete.
 Among the search algorithms that found optimal planning solution,
 the breadth first search performed best.

| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|-----------|-----------|--------------------|-------------|------------|
|BFS | 43 | 180 | 0.1745 | 6 | Optimal|
|BFTS | 1,458 | 5,960 | 6.0689 | 6 | optimal|
|DFGS | 12 | 48 | 0.0525 | 12 | Not optimal|
| Depth limited | 101 | 414 | 0.4362 | 50 | Not optimal|

##### Comparison of heuristics of A* search
All heuristics found the optimal solution.
The A* with h_pg_levelsum expanded smallest number of nodes,
though it took the longest time to run.
The h_ignore_preconditions reached goal state faster than
other two heuristics. Though the performance results of uninformed search
and heuristic search were quite similar for this small problem, T
he A* with h_ignore_preconditions performed slightly better than other search algorithms.


| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|-----------|-----------|--------------------|-------------|------------|
|A* with h_1| 55 | 224| 0.2438 |6| Optimal|
|A* with h_ignore_preconditions|41 |170| 0.1670| 6 |Optimal|
|A* with h_pg_levelsum |32 | 148 |1.3353 |6| Optimal|


# Problem 2
For this problem, additional cargo, plane, and airport are introduced.
Initially, the cargo 1 and plane 1 are at SFO; the cargo 2 and plane 2 are at JFK;
 and the cargo 3 and plane 3 are at ATL. At the goal state of the problem,
 the cargo 1 is at JFK, and cargo 2 and cargo 3 at ATL.
 Note that the goal state does not count where the planes end up.
 The optimal planning solution in this case consists of a sequence of 9 actions,
 loading the cargo at each airport (3), flying the planes to the desired airports(3),
 then unloading the cargos (3). An example of optimal planning is
 \[Load(C2, P2, JFK), Load(C1, P1, SFO), Load(C3, P3, ATL), Fly(P2, JFK, SFO),
 Unload(C2, P2, SFO), Fly(P1, SFO, JFK), Unload(C1, P1, JFK), Fly(P3, ATL, SFO),
 Unload(C3, P3, SFO)].

##### Comparison of uninformed search for three problems
Similar to problem 1, depth first graph search could find the solution quickly
and opened less nodes, but the solution found was not optimal.
Depth limited tree search also returned a nonoptimal solution after
expanding so many nodes. Breadth first tree search took more than reasonable time
 to find the solution in the local environment.
 I suspect this is because, unlike the graph search,
  the breadth first tree search does not exclude reversing to the previous state.
  For example, for tree search, repeating loading a cargo to a plane and unloading
  the same cargo right after is considered as executable actions.
Among the uninformed search algorithms, the BFS performed the best.
 It returned the optimal solution fast, expanding less number of nodes.

| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|----------- |-----------|--------------------|-------------|------------|
|BFS | 3,343 | 30,509 |65.8927 |9| Optimal|
|BFTS |- | -          |>10min  | - |-      |
|DFGS | 582| 5,211| 14.5242| 575| Not optimal
| Depth limited | 222,719 | 2,054,119| 4797.7268 |50| Not optimal|


##### Comparison of heuristics of A* search
Again, all heuristics found the optimal solution.
The h_pg_levelsum heuristic expanded significantly less number of new nodes,
 though it spent more time than other two.
 Depending on the user’s need for storage and runtime,
 the decision for the best heuristics could be different.
 If finding solution in quick runtime is important,
 then A* with h_ignore_preconditions is better, vice versa.

| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|-----------|----------- |--------------------|-------------|------------|
|A* with h_1| 4,853     |44,041      |80.8756             | 9 |Optimal|
|A* with h_ignore_preconditions|1,450 | 13,303 |24.0998 |9| Optimal|
|A* with h_pg_levelsum |168| 1,618| 110.4581| 9| Optimal|


### Problem 3
In this case, a total of four cargos are located at four
different airports and only two planes are available.
 Optimal planning solution has a sequence of 12 actions.
 One example of optimal solution is \[Load(C1, P1, SFO), Fly(P1, SFO, ATL), Load(C3, P1, ATL), Fly(P1, ATL, JFK), Load(C2, P1, JFK), Unload(C1, P1, JFK), Unload(C3, P1, JFK), Fly(P1, JFK, ORD), Load(C4, P1, ORD), Fly(P1, ORD, SFO), Unload(C2, P1, SFO), Unload(C4, P1, SFO)].



##### Comparison of uninformed search for three problems

The breadth first tree search and depth limited search could not find the solution in a reasonable time. Similar to the two previous problems, the solution found by depth first tree search was not optimal. Among the algorithms that found the optimal solution, BFS performed better. It was faster to run and opened less new nodes.

| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|----------- |-----------|--------------------|-------------|------------|
|BFS        |  9,456     |    71,407 |216.99247           | 12          |Optimal
|BFTS       |- | -          |>10min  | - |-      |
|DFGS | 832 | 3,692 |17.4152 |662 |Not optimal|
| Depth limited |- | -          |>10min  | - |-      |

##### Comparison of heuristics of A* search

A* with h_pg_levelsum heuristic took more than 10 minutes to run in the local system. A* with h_ignore_preconditions could find the solution very fast, however, the solution found was not optimal.

| Algorithm | Expansions | New nodes | Time elapsed (sec) | Plan length | Optimality |
|-----------|-----------|----------- |--------------------|-------------|------------|
|A* with h_1| 11,804    |88,393      |228.3863 |12 |Optimal|
|A* with h_ignore_preconditions|4,643 |36,113 |90.2555 |13| Not Optimal|
|A* with h_pg_levelsum |- |-| > 10m |-| -|




# Reference
- Artificial Intelligence: A Modern Approach, 3rd Ed, S. Russel and P. Norvig (2010)
- [MIT OpenCourseWare lecture on GraphPlan and making planning graphs](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-825-techniques-in-artificial-intelligence-sma-5504-fall-2002/lecture-notes/Lecture12FinalPart1.pdf)
