Here's a complete and clear graphic explanation of tree data structures with all key terminology:

```
                   Root
                    1
                  /   \
              2          3
           /    \         \
         4      5         6
        /               /    \
      7               8      9
```

### Full, Perfect Tree Terminologies Explained

- **Node**: Each circle with a number (e.g., 1, 2, 3...) is a node, the basic element that holds data.
- **Root**: The top node (1) from which all others descend. Every tree has exactly one root.
- **Edge**: The line connecting two nodes, showing relationships (e.g., edge between 1 and 2).
- **Parent**: A node that has one or more child nodes. For example, node 1 is the parent of 2 and 3.
- **Child**: A node that descends from a parent. Node 2 is a child of node 1.
- **Siblings**: Nodes sharing the same parent. Nodes 2 and 3 are siblings.
- **Leaf (External) Node**: Nodes with no children (7, 5, 8, 9) are leaves or terminal nodes.
- **Internal Node**: Nodes with at least one child (1, 2, 3, 4, 6) are internal nodes.
- **Subtree**: A tree formed from a node and all its descendants. For example, node 2 and its descendants (4, 5, 7) form a subtree.
- **Path**: A sequence from one node to another following edges. For example, path from 1 to 7 is [1 → 2 → 4 → 7].
- **Height of a Node**: The number of edges from that node down to its furthest leaf. For example, height of node 2 is 2 (to leaf node 7).
- **Depth of a Node**: The number of edges from the root to that node. Node 7 has depth 3.
- **Level**: Depth + 1 (root is at level 1, children of root at level 2, etc.).
- **Ancestor**: A node that appears on the path from the root to a given node (e.g., ancestors of 7 are 4, 2, 1).
- **Descendant**: Nodes coming from a node downwards (e.g., descendants of 2 are 4, 5, 7).
- **Degree of a Node**: Number of children a node has. Node 1 has degree 2.
- **Degree of a Tree**: Highest degree of any node in the tree.
- **Binary Tree**: A tree where every node has at most 2 children.
- **Full Binary Tree**: Each node has either 0 or 2 children.
- **Complete Binary Tree**: All levels except possibly the last are fully filled, and nodes are as left as possible.

This picture and terms cover the essential concepts for understanding tree data structures, useful for programming and algorithm study.[2][3][6][11]

