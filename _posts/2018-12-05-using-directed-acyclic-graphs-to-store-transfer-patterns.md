---
title: Using directed acyclic graphs to store transfer patterns
date: 2018-12-05 05:00
lastmod: 2018-12-05 05:00
description: >
  Different approaches to storing and merging directed acyclic graphs.
layout: post
---

Directed acyclic graphs (DAGs) are a data structure that show up in all sorts of fields: [distributed ledgers](https://www.iota.org/), [word counting](https://en.wikipedia.org/wiki/Deterministic_acyclic_finite_state_automaton) and [journey planning](https://ad.informatik.uni-freiburg.de/files/transferpatterns.pdf). One of the properties that make them useful is they can represent many variations using a small amount of memory:

![directed-acyclic-graph](/asset/img/directed-acyclic-graphs/directed-acyclic-graph.svg){: .center }

In fact, a tree structure is just a DAG where the parent has been restricted to a single node:

![tree](/asset/img/directed-acyclic-graphs/tree.svg){: .center }

# Transfer patterns

Directed acyclic graphs are particularly relevant to journey planning as they are very effective at storing paths. Hannah Bast's paper on [transfer patterns](https://ad.informatik.uni-freiburg.de/files/transferpatterns.pdf) describes an algorithm where the points that a passenger needs to change service are pre-processed in order to speed up real-time queries. Naively, this could be stored as a a key-value pair where the key is the journey origin and destination, and the value is a list of transfer patterns:

```
A->E = [A->B->C->D->E, A->C->E]
A->C = [A->B->C, A->C]
```

This approach is simple but not memory efficient. Travelling from A to E and travelling from A to C share much of the same information. Stored as a tree this information becomes:

```
    A
   / \
  B   C
  |   |
  C   E
  |
  D
  |
  E
```

Using this tree structure it is possible to compute the path from A->E and A->C using the same data structure. As nodes in a DAG can have multiple parents it can be compressed further:

```
  A
  | \
  B |
  | /
  C
  | \
  D |
  | /
  E
```

But note that the two structures are not logically equivalent. In the first the paths from A->E are:

```
A->E = [
  A->B->C->D->E,
  A->C->E
]
```

In the second there are more options:

```
A->E = [
  A->B->C->D->E,
  A->B->C->E,
  A->C->D->E,
  A->C->E
]
```

It's appropriate to use a DAG to store transfer patterns as any path to a node may be optimal, therefore any subsequent nodes may need that path. For example, if either A->C or A->B->C may be fastest and travelling to E requires a path via C, both A->C and A->B->C are potentially valid for any path to E.

There is no strict definition for the best way to represent a DAG, but using JSON it would become:

```
{
  "A": ["B", "C"],
  "B": ["C"],
  "C": ["D", "E"],
  "D": ["E"],
  "E": []
}
```

Alternatively, it can be represented as a [doubly-linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) where the next and previous node is an array of elements:

```
{
  "A": { "previous": [], "next": ["B", "C"] },
  "B": { "previous": ["A"], "next": ["C"] },
  "C": { "previous": ["A", "B"], "next": ["D", "E"] },
  "D": { "previous": ["C"], "next": ["E"] },
  "E": { "previous": ["C", "D"], "next": [] }
}
```

# Extracting paths from a DAG

How the data structure is stored will depend on the application. In this case it's beneficial to store it as a doubly linked list so the path to the specific node being requested can be extracted efficiently. Using the simplified JSON structure would require a [breadth](https://en.wikipedia.org/wiki/Breadth-first_search) or [depth](https://en.wikipedia.org/wiki/Depth-first_search) first search from the root node (origin) to the destination. As the location of the destinations in the graph are not known, a full scan of the entire graph is required.

Using the doubly linked list it is possible to go directly to the leaf node (the destination) and follow all paths back to the root. As the graph is acyclic reaching the root node is guaranteed in the most efficient time possible.

```
function getPaths(graph, path, origin, current) {
  // put the current node at the head of the list
  path.unshift(current);

  if (current === origin) {
    return [path];
  }
  else {
    const paths = [];

    // return paths to all parent nodes until the root node is reached
    for (const previous of graph[current].previous) {
      paths.push(...getPaths(graph, path.slice(), origin, previous));
    }

    return paths;
  }
}

const paths = getPaths(graph, [], "A", "E");

/* paths = [
 *   ["A", "C", "E"],
 *   ["A", "B", "C", "E"],
 *   ["A", "C", "D", "E"],
 *   ["A", "B", "C", "D", "E"]
 * ]
```

In our use case the `next` property is not used, so it is possible to revert to the simplified data structure using the list items as references to the parent, rather than leaf nodes:

```
{
  "A": [],
  "B": ["A"],
  "C": ["A", "B"],
  "D": ["C"],
  "E": ["C", "D"]
}
```

# Merging Directed Acyclic Graphs

During the generation of transfer patterns it's necessary to merge new patterns into the tree. There's no standard algorithm to merge DAGs because the algorithm will depend on the data structure used to store the DAG. For the example above, it's a simple case of iterating each node in the graph and merging array into a unique set of nodes.

```
const graphA = {
  "A": [],
  "B": ["A"],
  "C": ["B"],
  "D": ["C"],
  "E": ["D"]
}

const graphB = {
  "A": [],
  "C": ["A"],
  "E": ["C"]
}

// merges graphB into graphA
function mergeGraph(graphA, graphB) {
  for (const node in graphB) {
    // ensure the node exists in the target graph
    graphA[node] = graphA[node] || [];

    // insert every parent node from graphB that is not already in graphA
    const parents = graphB[node].every(n => !graphA[node].includes(n));
    graphA[node].push(...parents);
  }
}
```

Depending on the language used to implement the algorithm it may be more efficient to use a `Set` to store the next and previous values.

# DAGs

DAGs are a useful data structure that are a natural fit for transfer patterns. Each station can have it's own transfer pattern graph that is used to look up how to get to every other station in the network. In theory, it is possible to store the whole graph for every station in a single graph but this would need to be cyclic. In essence it would be a direct representation of the network topology and it would not be possible to run an efficient algorithm to look up a path between two stations.
