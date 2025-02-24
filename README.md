# graph

A library that can be used as a building block for high-performant graph
algorithms.

Graph provides implementations for directed and undirected graphs. Graphs
can be created programatically or read from custom input formats in a
type-safe way. The library uses [rayon](https://github.com/rayon-rs/rayon)
to parallelize all steps during graph creation.

The implementation uses a Compressed-Sparse-Row (CSR) data structure which
is tailored for fast and concurrent access to the graph topology.

**Note**: The development is mainly driven by
[Neo4j](https://github.com/neo4j/neo4j) developers. However, the library is
__not__ an official product of Neo4j.

## What is a graph?

A graph consists of nodes and edges where edges connect exactly two nodes. A
graph can be either directed, i.e., an edge has a source and a target node
or undirected where there is no such distinction.

In a directed graph, each node `u` has outgoing and incoming neighbors. An
outgoing neighbor of node `u` is any node `v` for which an edge `(u, v)`
exists. An incoming neighbor of node `u` is any node `v` for which an edge
`(v, u)` exists.

In an undirected graph there is no distinction between source and target
node. A neighbor of node `u` is any node `v` for which either an edge `(u,
v)` or `(v, u)` exists.

## How to use graph?

The library provides a builder that can be used to construct a graph from a
given list of edges.

For example, to create a directed graph that uses `usize` as node
identifier, one can use the builder like so:

```rust
use graph::prelude::*;

let graph: DirectedCsrGraph<usize> = GraphBuilder::new()
    .edges(vec![(0, 1), (0, 2), (1, 2), (1, 3), (2, 3)])
    .build();

assert_eq!(graph.node_count(), 4);
assert_eq!(graph.edge_count(), 5);

assert_eq!(graph.out_degree(1), 2);
assert_eq!(graph.in_degree(1), 1);

assert_eq!(graph.out_neighbors(1), &[2, 3]);
assert_eq!(graph.in_neighbors(1), &[0]);
```

To build an undirected graph using `u32` as node identifer, we only need to
change the expected types:

```rust
use graph::prelude::*;

let graph: UndirectedCsrGraph<u32> = GraphBuilder::new()
    .edges(vec![(0, 1), (0, 2), (1, 2), (1, 3), (2, 3)])
    .build();

assert_eq!(graph.node_count(), 4);
assert_eq!(graph.edge_count(), 5);

assert_eq!(graph.degree(1), 3);

assert_eq!(graph.neighbors(1), &[0, 2, 3]);
```

It is also possible to create a graph from a specific input format. In the
following example we use the `EdgeListInput` which is an input format where
each line of a file contains an edge of the graph.

```rust
use std::path::PathBuf;

use graph::prelude::*;

let path = [env!("CARGO_MANIFEST_DIR"), "resources", "example.el"]
    .iter()
    .collect::<PathBuf>();

let graph: DirectedCsrGraph<usize> = GraphBuilder::new()
    .csr_layout(CsrLayout::Sorted)
    .file_format(EdgeListInput::default())
    .path(path)
    .build()
    .expect("loading failed");

assert_eq!(graph.node_count(), 4);
assert_eq!(graph.edge_count(), 5);

assert_eq!(graph.out_degree(1), 2);
assert_eq!(graph.in_degree(1), 1);

assert_eq!(graph.out_neighbors(1), &[2, 3]);
assert_eq!(graph.in_neighbors(1), &[0]);
```

## Examples?

Check the [TriangleCount](./examples/triangle_count.rs) and
[PageRank](./examples/page_rank.rs) implementations  to see how the library
is used to implement high-performant graph algorithms.

License: MIT
