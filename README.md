[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/ulyILqqB)
# Weekly Coding #9: Midnight Monster Delivery

## Summary

This program finds the cheapest delivery routes through a haunted city graph. Each location is a node and each haunted road is a directed, weighted edge with a positive travel cost. The core algorithm is Dijkstra's algorithm implemented with Python's `heapq` min-heap as the priority queue. The program can find the cheapest cost to every reachable location from a starting point, reconstruct the exact shortest path between two locations, and pick the cheapest first delivery among multiple targets.

## Approach

- **Graph representation**: adjacency dictionary — each key is a location name, each value is a dict mapping neighbor names to integer travel costs.
- **Priority queue / frontier**: a `heapq` min-heap storing `(cost, node)` tuples. The smallest cost is always popped first, which is what makes Dijkstra's correct.
- **Relaxation**: when we pop a node, we check each neighbor. If `current_cost + edge_weight < known_distance[neighbor]`, we update `distances[neighbor]` and push the new `(cost, neighbor)` onto the heap. Stale (outdated) heap entries are skipped by comparing the popped cost to the currently known best distance.
- **Path reconstruction**: `shortest_monster_delivery` keeps a `previous` map alongside `distances`. When we find a cheaper path to a neighbor, we record `previous[neighbor] = current_node`. After reaching the target, we walk backwards through `previous` from `target` to `start`, then reverse the list to get the path in the correct order.

## Complexity

```text
Time complexity:  O((V + E) log V)
Space complexity: O(V) extra space, or O(V + E) including graph storage
```

- **`monster_delivery_costs`**:
  - Time: O((V + E) log V) — each node is settled once; each edge relaxation may push to the heap (O(log V) per push); total heap operations ≤ E.
  - Space: O(V) for the `distances` dict and the heap (at most E entries in worst case, but bounded by O(V + E)).
  - Why: The heap ensures we always process the cheapest unsettled node first. The stale-entry check avoids re-processing without needing a decrease-key operation.

- **`shortest_monster_delivery`**:
  - Time: O((V + E) log V) — same as above; we add an early exit when the target is settled, which can save work in practice but doesn't change worst-case complexity.
  - Space: O(V) extra for `distances`, `previous`, and the heap.
  - Why: The `previous` map adds only O(V) extra space. Path reconstruction is O(V) since the path length is at most V nodes.

## Edge-Case Checklist

- [x] start equals target — returns `(0, [start])` immediately
- [x] target is unreachable — returns `(inf, [])`
- [x] start node is missing — `monster_delivery_costs` raises `ValueError`; `shortest_monster_delivery` returns `(inf, [])`
- [x] target node is missing — returns `(inf, [])`
- [x] node has no outgoing edges — treated correctly; the loop over neighbors simply does nothing
- [x] graph contains cycles — stale-entry skip (`if current_cost > distances[node]: continue`) prevents infinite loops
- [x] tied shortest paths — either valid path is accepted; the heap's insertion order determines which is found first
- [x] negative edge weight — caught by `validate_haunted_map`, raises `ValueError`
- [x] zero edge weight — caught by `validate_haunted_map`, raises `ValueError`
- [x] neighbor not listed as a graph node — caught by `validate_haunted_map`, raises `ValueError`

## Tests I Added

- `test_best_next_monster_stop_picks_cheapest` — verifies the stretch function returns the lowest-cost reachable target
- `test_best_next_monster_stop_tie_picks_first` — verifies tie-breaking favors the earlier entry in the `targets` list
- `test_best_next_monster_stop_no_reachable` — verifies `("", inf)` is returned when no target is reachable
- `test_best_next_monster_stop_missing_start` — verifies `("", inf)` is returned when `start` is not in the graph

## Assistance & Sources

AI used? Y

- Helped structure and verify the implementation and README. All algorithmic decisions (Dijkstra's with heap, stale-entry skip, previous-map path reconstruction) follow standard computer science literature.

Other sources used:

- Python docs: `heapq` — https://docs.python.org/3/library/heapq.html
- Dijkstra's algorithm — CLRS *Introduction to Algorithms*, Chapter 24

## Notes for Instructor

The stretch function `best_next_monster_stop` is fully implemented. It runs Dijkstra once from `start` to get all costs in O((V + E) log V), then scans `targets` in order in O(T) where T = len(targets). Ties are broken by list order because we only update `best` when strictly less than the current best cost.