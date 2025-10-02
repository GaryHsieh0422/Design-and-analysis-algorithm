# Design and analysis algorithm
# Lecture 2: Greedy Algorithms

## Introduction to Greedy Algorithms
- A technique to solve problems by always making the locally best choice at the moment.
- Hopefully, a series of locally best choices leads to a globally best solution.
- Greedy algorithms yield optimal solutions for many (but not all) problems.

## Interval Scheduling
### Problem Definition
- Input: n jobs, each job j starts at s_j and finishes at f_j.
- Two jobs are compatible if they do not overlap.
- Goal: Find the maximum subset of mutually compatible jobs.
- Example: Jobs labeled a to h on a timeline from 0 to 11.
  - {a, g} is a compatible set.
  - {b, e, h} is another compatible set.

### Ideas for Greedy Rules
- Choose the interval that starts earliest (rationale: start using resource ASAP).
- Choose the smallest interval (rationale: fit more small jobs).
- Choose the interval that overlaps with the fewest remaining intervals (rationale: keep options open).

### Rules That Do Not Work
- Earliest start time: May select overlapping jobs inefficiently (e.g., selects short overlapping segments, missing longer optimal sets).
- Shortest job: May prioritize tiny jobs that block larger sets (e.g., selects a long central job, but optimal is multiple shorts around it).
- Fewest conflicts: May not maximize overall (e.g., selects a job with 2 conflicts, but optimal green solution selects more).
- Green solutions in examples are optimal; these rules fail to achieve them.

### Effective Greedy Rule: Earliest Finish Time
- Sort jobs by increasing finish time: f1 ≤ f2 ≤ ... ≤ fn.
- Select each job if compatible with previously selected ones (s_j ≥ f_j* where j* is last selected).
- Example:
  - Jobs (s, f): (0,10), (3,4), (2,8), (1,5), (4,5), (4,8), (5,6), (7,9).
  - Sorted by f: (3,4), (1,5), (4,5), (5,6), (4,8), (2,8), (7,9), (0,10).
  - Selected: (3,4), (4,5), (5,6), (7,9).

### Greedy Algorithm Pseudocode
```
Sort jobs by finish times so that f1 ≤ f2 ≤ ... ≤ fn.
A ← ∅
for j = 1 to n {
    if (job j compatible with A)  // s_j ≥ f_j* (j* = last in A)
        A ← A ∪ {j}
}
return A
```
- Time complexity: O(n log n) due to sorting.
- Compatibility check: Remember last job's finish time.

### Proof of Correctness (Exchange Argument)
- Let G = i1, i2, ..., ik be greedy solution.
- Let OPT = j1, j2, ..., jm be an optimal solution (assume sorted by finish time).
- Assume i1 = j1, ..., ir = jr, but ir+1 ≠ jr+1 (r ≥ 0).
- ir+1 finishes ≤ jr+1 (by greedy choice).
- Replace jr+1 with ir+1 in OPT to get OPT' (still optimal, as ir+1 ends earlier or same, compatible).
- Now G and OPT' agree on first r+1 jobs.
- Repeat until G = OPT or |G| = |OPT|, proving optimality.
- Remarks:
  - |A| = |OPT| is weaker than A = OPT.
  - Multiple OPTs exist, e.g., {(0,1),(1,2)} and {(0,1),(1,3)} for input (0,1), (1,2), (1,3).

### Alternative Proof (Without Figures)
- Let G = i1 i2 ... im (greedy, sorted by finish).
- Let OPT = j1 j2 ... jk (optimal).
- If i1 ≠ j1, replace j1 with i1 (i1 finishes ≤ j1, still optimal).
- Update OPT to i1 j2 ... jk.
- Repeat for i2, etc.
- Eventually OPT becomes G or has extra jobs (impossible, as no job starts after im's finish).
- Thus, |G| = |OPT|.

### Example Schedules
- Various selections shown on timelines (pages 16-23), demonstrating greedy picks like B, then E, then H, etc.

## Interval Partitioning
### Problem Definition
- Input: n lectures, each j starts at s_j, finishes at f_j.
- Goal: Minimum number of classrooms so no two lectures overlap in the same room.
- Example: 10 lectures, suboptimal uses 4 rooms; optimal uses 3.

### Lower Bound
- Depth: Maximum number of overlapping lectures at any time.
- Classrooms needed ≥ depth (e.g., depth 3 requires at least 3).

### Greedy Algorithm
- Sort lectures by start time: s1 ≤ s2 ≤ ... ≤ sn.
- Assign each lecture to an existing classroom if possible (finish time of last in room ≤ s_j).
- Else, open new classroom.
- Use priority queue for classroom finish times.
Sort intervals by starting time so that s1 ≤ s2 ≤ ... ≤ sn.
d ← 0  // number of classrooms
for j = 1 to n {
    if (lecture j fits in existing classroom k)  // some k's last finish ≤ s_j
        assign to k, update k's finish
    else
        d ← d + 1
        assign to new classroom d
}
- Time complexity: O(n log n).

### Analysis
- Greedy never overlaps in same room (feasible).
- Proof of optimality (Structural Bound):
  - d = classrooms allocated.
  - When opening d, job j couldn't fit in d-1 rooms; all d-1 occupied by lectures starting ≤ s_j.
  - Thus, d lectures overlap at s_j + ε ⇒ d ≤ depth.
  - Any solution needs ≥ depth ⇒ greedy is optimal.
- Depth example: 3 (a,b,c overlap at 9:30).

## Greedy Analysis Strategies
- **Exchange Argument**: Transform any solution to greedy's without reducing quality.
- **Structural Bound**: Find bound every solution must meet; show greedy achieves it.

# Lecture 3: Minimum Spanning Tree (MST)
## Introduction to Graphs
- Graph: Contains subgraphs.
- Tree: (1) Connected, and (2) no cycle (equivalent to unique path between nodes or |edges| = |nodes| - 1).
- Forest: Disjoint union of trees (graph without cycles).

## Problem: Rail Network
- Connect towns with railways.
- Naïve approach: Connect all towns directly to a central hub (expensive, shown as star topology).
- Better approach: Minimize the total length of railways connecting all towns (forms a tree structure).

## Definition of MST
- Spanning tree: Subgraph T of undirected connected graph G=(V, E) that is a tree and contains all nodes of G.
- Minimum Spanning Tree (MST): Spanning tree with minimum total weight (cost or length) among edges.
- Example: In a graph with nodes a,b,c,d and edges of weights 1,2,3,4, there are 16 spanning trees; MSTs have total weight 6 (two possible MSTs shown).

## Generic MST Algorithm
- Set A is always a subset of some MST (invariant property).
- Safe edge (u,v): Adding it to A does not destroy the invariant.
- Pseudocode:
  ```
  GENERIC_MST(G, w)
  1 A := {}
  2 while A does not form a spanning tree do
  3   find an edge (u,v) that is safe for A
  4   A := A ∪ {(u,v)}
  5 return A
  ```

## Safe Edge Concepts
- Cut (S, V-S): Partition of V.
- Edge crosses cut if endpoints in different partitions.
- Light edge: Shortest weight among edges crossing the cut.
- Theorem: If A is subset of MST, no edge in A crosses cut (S,V-S), and (x,y) is light edge crossing cut, then (x,y) is safe for A.
  - Proof outline: Consider MST T_opt including A. If (x,y) in T_opt, safe. Else, replace another crossing edge in T_opt with (x,y) to get another MST including A ∪ {(x,y)}.
- Corollary: If C is connected component in forest G_A=(V,A), and (u,v) is light edge connecting C to another component, then (u,v) is safe (basis for Kruskal).

## Kruskal's Algorithm
- Builds MST by adding shortest edges that do not form cycles.
- Uses disjoint-set (union-find) for efficiency.
- Basic pseudocode:
  ```
  1 (Sort edges in increasing order)
  2 A := {}
  3 while E is not empty do {
  4   take shortest edge (u,v) and delete from E
  5   if u and v in different components then
  6     add (u,v) to A
  7 }
  ```
- Full pseudocode with union-find:
  ```
  MST_KRUSKAL(G,w)
  1 A := {}
  2 for each vertex v in V[G]
  3   do MAKE_SET(v)
  4 sort edges of E by nondecreasing weight w
  5 for each edge (u,v) in E, in nondecreasing order
  6   do if FIND_SET(u) != FIND_SET(v)
  7     then A := A ∪ {(u,v)}
  8          UNION(u,v)
  9 return A
  ```
- Disjoint-set operations: MAKE_SET, UNION, FIND_SET (each O(log n)).
- Running time: O(|E| log |E|).
- Example: Step-by-step on 10-node graph (A-J) with edges of various weights, adding non-cycle-forming edges in order (images show progressive addition and cycle detection).

## Prim's Algorithm
- Grows MST from a root r, adding shortest edge to non-tree vertex.
- Uses priority queue for efficiency.
- Basic pseudocode:
  ```
  MST_PRIM(G,w,r)
  1 A := {}
  2 S := {r}
  3 Q := V - {r}
  4 while Q is not empty do {
  5   take edge (u,v) where u ∈ S, v ∈ Q, and (u,v) shortest such
  6   add (u,v) to A, add v to S, delete v from Q
  7 }
  ```
- Full pseudocode:
  ```
  MST_PRIM(G, w, r)
  1 for each v in V do
  2   key[v] := ∞, parent[v] := NIL
  3 A := ∅
  4 key[r] := 0; parent[r] := NIL;
  5 Q ← (V, key)  /* initialize priority queue Q */
  6 while Q != {} do
  7   u := EXTRACT_MIN(Q); if parent[u] ≠ NIL, A := A ∪ {(u, parent[u])}
  8   for each v in Adj[u] do
  9     if v in Q and w(u,v) < key[v]
  10      then parent[v] := u
  11           key[v] := w(u,v)
  12           Update(v, key[v])
  ```
- Key[v]: Shortest distance from tree to v.
- Parent[v]: Parent in tree.
- Q: Priority queue of non-tree vertices, keyed by key[].
- Operations: EXTRACT_MIN, Update (O(log |V|)).
- Running time: O(|E| log |V|); with Fibonacci heap, O(|E| + |V| log |V|).
- Example: Step-by-step on graph, with tables showing key and parent updates (e.g., starting from A, adding C, then B, etc.).

## Summary
- MST: Spanning tree with minimum total edge weight.
- Cut and safe edge concepts.
- Generic MST algorithm.
- Kruskal's algorithm: Sort edges, add if no cycle (union-find).
- Prim's algorithm: Grow from root, add min-edge to fringe (priority queue).

## Challenge Problems
- Compute maximum spanning tree: Do Prim and Kruskal work if choosing max-cost crossing edge?
- Prove: If all edge weights distinct, MST is unique.

# Lecture 4: Single-Source Shortest Paths
## Problem Definition
- Given a directed graph G=(V, E, W) where each edge has a weight (length, cost).
- Find a shortest path from source s to destination v.
- Example: Finding a route on a map from point A to B requires careful algorithm design.

## Paths in Graphs
- The total weight (length) of a path p = v1 → v2 → … → vk is ∑_{i=1}^{k-1} w(vi, vi+1).
- Example: Path with weights 4, 2, 5, 1 has total weight 12.

## Shortest Paths
- A shortest path from u to v is a path of minimum total weight.
- Shortest-path weight δ(u, v) = min {w(p) : p is a path from u to v}.
- δ(u, v) = +∞ if no path exists.

## Well-Definedness of Shortest Paths
- If the graph contains a negative-weight cycle, some shortest paths may not exist (e.g., infinite reduction by cycling).
- Negative-weight cycles: Total weight in the cycle is negative.
- If no negative-weight cycles reachable from s, δ(s, v) is well-defined, even if negative.
- If a negative-weight cycle reachable from s to v, δ(s, v) = -∞.
- Assumption: All edge weights ≥ 0.

## Optimal Substructure
- Theorem: A subpath of a shortest path must be a shortest path.
- Proof: If a subpath (w → x) is not shortest, replacing it with a shorter one yields a shorter overall path, contradicting the assumption.

## Representing Shortest Paths
- Maintain for each v ∈ V:
  - π[v]: Predecessor of v (node before v in the shortest path).
  - d[v]: Current shortest path length from s to v.
- Backtracking using π[] reconstructs the path (e.g., from v back to s).
- Backtracking code:
  ```
  print (u)
  x = π(u)
  while (x ≠ s)
      print (x)
      x = π(x)
  print x
  ```

## Observation
- Suppose a shortest path p from s to v decomposes as s → … → u → v.
- Then δ(s, v) = δ(s, u) + w(u, v).
- u is unknown but can try all in V in O(n) time.
- If no u, direct edge (s, v) is shortest.
- First shortest from s to some node: Shortest edge from s (greedy).

## Relaxation
- Process of relaxing edge (u, v): Test if path through u improves d[v]; update if so.
- RELAX(u, v, W):
  ```
  if d[v] > d[u] + w(u, v)
      then d[v] ← d[u] + w(u, v)
           π[v] ← u
  ```
- Examples: (a) d[v] decreases if improved; (b) unchanged if not.

## Dijkstra's Algorithm (Greedy)
- Maintain S (nodes with finalized shortest paths), V-S, π, d.
- S: Set of nodes with known shortest paths from s.
- For every v ∈ V, π[v] ∈ S or NIL.
- π[v]: Node before v in current shortest path (using only S intermediates).
- d[v]: Current shortest path length from s to v (using only S intermediates).
- At each step:
  - Add v from V-S with smallest d[v] to S (greedy on d[]).
  - Declaim d[v] as final shortest path length.
  - Update d and π for all nodes in V-S adjacent to v (relaxation).
- Initialization:
  ```
  INITIALIZE-SINGLE-SOURCE(G, s)
  for each v ∈ V\{s}
      do d[v] ← ∞
         π[v] ← NIL
  d[s] ← 0
  ```
- Algorithm:
  - Assumes w(e) ≥ 0 for all edges.
  - Maintain S where for v ∈ S, d[v] = δ(s, v).
  - (a) Select u ∈ V-S with d[u] = min {d[x] | x ∈ V-S}; S ← S ∪ {u}.
  - (b) For each v adjacent to u, RELAX(u, v, W).
  - Repeat until S = V.
- Pseudocode:
  ```
  d[s] ← 0
  for each v ∈ V – {s}
      do d[v] ← ∞, π[v] ← NIL.
  S ← ∅
  Q ← V  % Q is a priority queue maintaining V – S
  while Q ≠ ∅
      do u ← EXTRACT-MIN(Q)
         S ← S ∪ {u}
         for each v ∈ Adj[u]
             do if d[v] > d[u] + w(u, v)
                 then d[v] ← d[u] + w(u, v), π[v] ← u
  ```
- Example: Step-by-step on graph with nodes A,B,C,D,E and edges with weights 10,3,1,4,7,9,8,2,2; shows updates to d, π, S, Q.

## Implementation
- Priority queue Q stores V-S, keyed by d[] values.
- Graph represented by adjacency lists.

## Time Complexity
- Depends on priority queue implementation.
- Method 1 (array):
  - EXTRACT-MIN: O(|V|), total O(|V|^2).
  - RELAX: O(1), total O(|E|).
  - Total: O(|V|^2).
- Method 2 (adaptable heap):
  - EXTRACT-MIN and RELAX: O(log |V|).
  - Total: O(|E| log |V|).
  - Faster for sparse graphs (|E| = O(|V|)).

## Proof of Correctness
- Theorem: For S at any time, for each v ∈ S, d[v] is the shortest s-v path length.
- Proof by induction on |S|:
  1. |S|=1 (S={s}): d[s]=0 holds.
  2. Assume true for |S|=k.
  3. Add v with smallest d[v] to make |S|=k+1.
     - Let P be any s-v path.
     - Case 1: All before v in S: Length(P) ≥ d[v] (by min over predecessors).
     - Case 2: P crosses S to V-S at x to y: Length(P) ≥ d[y] ≥ d[v] (by greedy choice and non-negative edges).
     - Thus, d[v] is shortest.

## Limitations
- Does not work with negative-weight edges.
- Example: Graph with edges s→u (2), s→v (1), u→v (-10); s→v (1) selected first, misses shorter s→u→v (-8).
