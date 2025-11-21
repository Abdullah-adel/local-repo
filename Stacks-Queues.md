### Stack = Last In, First Out (LIFO)

- Think of a stack of plates:
  - Add plates on top (push)
  - Remove plates from top (pop)
  - Can only access the top plate

- Basic operations:
  - **Push**: Add element to top
  - **Pop**: Remove element from top
  - **Peek/Top**: Look at top element without removing
  - **isEmpty**: Check if stack is empty


<img width="1086" height="752" alt="image" src="https://github.com/user-attachments/assets/541e7c7e-e2cf-4002-9068-6cf56966bd66" />

Example:
 - Push 1: [1]
 - Push 2: [1, 2]
 - Push 3: [1, 2, 3]
 - Pop:    [1, 2]     (removed 3)
 - Pop:    [1]        (removed 2)


Python implementation:
```
stack = []
stack.append(5)    # Push
stack.append(10)   # Push
top = stack.pop()  # Pop (returns 10)
```

Common uses:
- Function calls (call stack)
- Undo operations
- Browser back button
- Expression evaluation
- Backtracking algorithms

Key point: You can only access the most recently added item.
