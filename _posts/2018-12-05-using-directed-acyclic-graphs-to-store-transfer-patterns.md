---
title: Using directed acyclic graphs to store transfer patterns
date: 2018-12-10 05:00
lastmod: 2018-12-10 05:00
description: >
  Different approaches to storing and merging directed acyclic graphs.
layout: post
---

Directed acyclic graphs (DAGs) are a data structure that show up in all sorts of fields: [distributed ledgers](https://www.iota.org/), [word counting](https://en.wikipedia.org/wiki/Deterministic_acyclic_finite_state_automaton) and [journey planning](https://ad.informatik.uni-freiburg.de/files/transferpatterns.pdf). One of the properties that make them useful is they can represent many variations using a small amount of memory:

![directed-acyclic-graph](/asset/img/directed-acyclic-graphs/directed-acyclic-graph.svg){: .center }

In fact, the more common [tree structure](https://en.wikipedia.org/wiki/Tree_(data_structure)) is just a DAG where the parent has been restricted to a single node:

![tree](/asset/img/directed-acyclic-graphs/tree.svg){: .center }

# Transfer patterns

Directed acyclic graphs are particularly relevant to journey planning as they are very effective at storing paths. Hannah Bast's paper on [transfer patterns](https://ad.informatik.uni-freiburg.de/files/transferpatterns.pdf) describes an algorithm where the points that a passenger needs to change service are pre-processed in order to speed up real-time queries. Naively, this could be stored as a a key-value pair where the key is the journey origin and destination, and the value is a list of transfer patterns:

```
A->E = [A->B->C->D->E, A->C->E]
A->C = [A->B->C, A->C]
```

This approach is simple, but not memory efficient. Travelling from A to E and travelling from A to C share much of the same information. Stored as a tree this information becomes:

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

While this may not seem a problem at first, merging in a path from A to C, via D `[A, B, D, C]` creates a cycle as C has a parent of D and D has a parent of C. The directed acyclic graph is now just a directed graph. To avoid cyclic graphs each path can only be merged from the point it becomes identical to another path. Using our example above, adding `[A, B, D, C]` would share a path with `[A, B, C, D]` from B:

```
     A
    / \
   B   C
 / |   |
D  C   E
|  |
C  D
   |
   E
```

There is no strict definition for the best way to represent a DAG, as such it can be represented in many ways. A [tree](https://en.wikipedia.org/wiki/Tree_(data_structure)) can be used to represent a top down view, starting at the root node (A) and traversing down to the leaves:

```
const tree = [
  {
    label: "A",
    leaves: [
      {
        label: "B",
        leaves: [
          { label: "C", leaves: [ ... ] },
        ]
      },
      { label: "C", leaves: [ ... ] },
    ]
  }
]
```

Alternatively, it can be represented from the bottom up using a [linked list](https://en.wikipedia.org/wiki/Linked_list) where the next element is a reference to the parent node. With this method each path becomes it's own unique list, but elements of the list a re-used to save memory:

```
const pathFromB = {
  label: "B",
  next: {
    label: "A",
    next: null
  }
};

const firstFromD = {
  label: "D",
  next: {
    label: "C",
    next: pathFromB
  }
}

const secondFromD = {
  label: "D",
  next: pathFromB
}

```

The next element is the node label and index of the path in that nodes paths

# Extracting paths from a DAG

How the data structure is stored will depend on the application. In this case it's beneficial to store it as a linked list so the path to the specific node being requested can be extracted efficiently. Using the top down approach would require a [breadth](https://en.wikipedia.org/wiki/Breadth-first_search) or [depth](https://en.wikipedia.org/wiki/Depth-first_search) first search from the root node (origin) to the destination. As the location of the destinations in the graph are not known, a full scan of the entire graph is required.

Using the linked list it is possible to go directly to the leaf node (the destination) and follow all paths back to the root. As the graph is acyclic reaching the root node is guaranteed in the most efficient time possible.

```
// variables with & denote a reference
const graph = {
  "A": [{ label: "A", next: null }],
  "B": [{ label: "B", next: &A }],
  "C": [
    { label: "C", next: &B },
    { label: "C", next: &A }
  ],
  "D": [
    { label: "D", next: &C },
    { label: "D", next: &B }
  ],
  "E": [
    { label: "E", next: &D },
    { label: "E", next: &C1 }
  ]
}

function getPath(node, path) {
  path.unshift(node.label);

  return node.parent ? getPath(node.parent, path) : path;
}

const paths = graph["E"].map(node => getPath(node, []));

/* paths = [
 *   ["A", "C", "E"],
 *   ["A", "B", "C", "E"],
 *   ["A", "C", "D", "E"],
 *   ["A", "B", "C", "D", "E"]
 * ]
```

# Merging Directed Acyclic Graphs

During the generation of transfer patterns it's necessary to merge new patterns into the tree. There's no standard algorithm to merge DAGs because the algorithm will depend on the data structure used to store the DAG. For the example above, new paths are added to each node until a point is reached where the rest of the path already exists in the graph:

```
function merge([head, ...tail], results) {
  results[head] = results[head] || [];

  // search for an identical path in the tree
  let node = results[head].find(n => isSame(tail, n.parent));

  // if there isn't one
  if (!node) {
    // merge the tail of the path
    const parent = tail.length > 0 ? merge(tail, results) : null;

    // create a node maintaining a reference to the parent path
    node = { label: head, parent: parent };

    // add the node to result set for the stop "head"
    results[head].push(node);
  }

  return node;
}

function isSame(path, node) {
  for (let i = 0; node; i++, node = node.parent) {
    if (node.label !== path[i]) {
      return false;
    }
  }

  return true;
}
```

# Benefits

DAGs are a useful data structure that are a natural fit for transfer patterns. Each station can have it's own transfer pattern graph that is used to look up how to get to every other station in the network. In theory, it is possible to store the whole graph for every station in a single graph but this would need to be cyclic. In essence it would be a direct representation of the network topology and it would not be possible to run an efficient algorithm to look up a path between two stations.

Taking the UK rail network as an example, a single station's transfers pattern use roughly 12mb of memory when using a DAG and 32mb when storing as a plain key/value string. It should be noted that the storing references to variables is not possible in JSON and it's necessary to pre-process the DAG when loading it from disk or database.
