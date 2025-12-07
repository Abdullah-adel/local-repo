> Explain Recursion and rBST data structure simply and technically with use cases and o(n), share the most common 5 interview questions and 3 common interview coding with small context for each challenge related to Recursion and rBST with the ideal answers


> Recursion

Simple Explanation:
A function that calls itself to solve smaller versions of the same problem.

Key Components:
- **Base case**: Condition to stop recursion
- **Recursive case**: Function calls itself with modified input

Example:
```
def factorial(n):
    if n <= 1:        # Base case
        return 1
    return n * factorial(n-1)  # Recursive case
```

Time Complexity: Depends on problem - often O(2^n) for naive solutions, O(n) for linear recursion

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


BST (Binary Search Tree)

Simple Explanation:
Binary tree where left child < parent < right child for all nodes.

Properties:
- Left subtree values < root
- Right subtree values > root
- Both subtrees are also BSTs
```
Visual:
    8
   / \
  3   10
 / \    \
1   6    14
   / \   /
  4   7 13
```

Time Complexities:
- **Balanced BST**: Search/Insert/Delete O(log n)
- **Unbalanced BST**: O(n) worst case
- **Space**: O(n)

Use Cases:
- **Recursion**: Tree traversals, divide-and-conquer, backtracking
- **BST**: Databases, file systems, expression parsing, autocomplete

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


5 Most Common Interview Questions:

1. "What is recursion and when would you use it?"
   - **Answer**: "Function calling itself with smaller input. Use for problems with recursive structure like trees, divide-and-conquer, or when iterative solution 
is complex."

2. "What's the difference between recursion and iteration?"
   - **Answer**: "Recursion uses function calls and stack memory, iteration uses loops. Recursion is cleaner for tree problems but uses more memory."

3. "How do you avoid stack overflow in recursion?"
   - **Answer**: "Ensure proper base case, limit recursion depth, or convert to iterative solution using explicit stack."

4. "What makes a valid BST?"
   - **Answer**: "For every node: all left subtree values < node < all right subtree values. Not just immediate children."

5. "How do you traverse a BST and what are the orders?"
   - **Answer**: "Inorder (left-root-right) gives sorted order, preorder (root-left-right) for copying, postorder (left-right-root) for deletion."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


3 Common Coding Problems:

1. Validate Binary Search Tree
   - **Context**: Check if binary tree is valid BST
   - **Challenge**: Must check entire subtree ranges, not just immediate children

```
def isValidBST(root, min_val=float('-inf'), max_val=float('inf')):
    if not root:
        return True
    
    if root.val <= min_val or root.val >= max_val:
        return False
    
    return (isValidBST(root.left, min_val, root.val) and 
            isValidBST(root.right, root.val, max_val))
```

2. Binary Tree Maximum Path Sum
   - **Context**: Find path with maximum sum (can start/end at any node)
   - **Challenge**: Handle negative values and choose optimal path

```
def maxPathSum(root):
    max_sum = float('-inf')
    
    def max_gain(node):
        nonlocal max_sum
        if not node:
            return 0
        
        left_gain = max(max_gain(node.left), 0)
        right_gain = max(max_gain(node.right), 0)
        
        current_max = node.val + left_gain + right_gain
        max_sum = max(max_sum, current_max)
        
        return node.val + max(left_gain, right_gain)
    
    max_gain(root)
    return max_sum
```

3. Lowest Common Ancestor in BST
   - **Context**: Find lowest common ancestor of two nodes in BST
   - **Challenge**: Use BST property for efficient solution

```
def lowestCommonAncestor(root, p, q):
    if not root:
        return None
    
    # If both nodes are in left subtree
    if p.val < root.val and q.val < root.val:
        return lowestCommonAncestor(root.left, p, q)
    
    # If both nodes are in right subtree
    if p.val > root.val and q.val > root.val:
        return lowestCommonAncestor(root.right, p, q)
    
    # If nodes are on different sides, current node is LCA
    return root

```
These problems test core concepts: BST validation, recursive tree traversal with state, and leveraging BST properties for optimization.

