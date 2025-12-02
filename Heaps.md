> Explain heaps data structure simply and technically with use cases and o(n), share the most common 5 interview questions and 3 common interview coding with small context for each challenge related to heaps with the ideal answers

> Heaps Data Structure

Simple Explanation:
A heap is a complete binary tree where parent nodes follow a specific order rule with their children.

Types:
- **Max Heap**: Parent ≥ children (largest at root)
- **Min Heap**: Parent ≤ children (smallest at root)

Visual Example (Min Heap):
```
       1
      / \
     3   2
    / \ / \
   7  4 5  6
```

Technical Details:

Properties:
- Complete binary tree (filled left to right)
- Heap property maintained at all levels
- Usually implemented as array

Array Representation:
python
heap = [1, 3, 2, 7, 4, 5, 6]
# Parent at index i: children at 2i+1, 2i+2
# Child at index i: parent at (i-1)//2


Time Complexities:
- **Insert**: O(log n)
- **Extract min/max**: O(log n)
- **Peek min/max**: O(1)
- **Build heap**: O(n)
- **Heapify**: O(log n)

Space Complexity: O(n)

Use Cases:
- **Priority Queues**: Task scheduling, Dijkstra's algorithm
- **Heap Sort**: Efficient sorting algorithm
- **Finding K largest/smallest**: Top K problems
- **Median Finding**: Running median in data streams
- **Graph Algorithms**: Prim's MST, Dijkstra's shortest path

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


5 Most Common Heap Interview Questions:

1. "What's the difference between min heap and max heap?"
   - **Answer**: "Min heap: parent ≤ children, smallest at root. Max heap: parent ≥ children, largest at root. Both maintain heap property."

2. "How do you implement a heap using an array?"
   - **Answer**: "Parent at index i, children at 2i+1 and 2i+2. Child at index i, parent at (i-1)//2. Complete binary tree property maintained."

3. "What's the time complexity of heap operations?"
   - **Answer**: "Insert/delete: O(log n), peek: O(1), build heap: O(n). Height is log n due to complete tree structure."

4. "How does heapify work?"
   - **Answer**: "Bubble up for insert (compare with parent), bubble down for extract (compare with children). Maintains heap property."

5. "When would you use a heap vs other data structures?"
   - **Answer**: "Use heap for priority queues, finding extremes, when you need O(1) access to min/max but don't need full sorting."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━


3 Common Heap Coding Problems:

1. Kth Largest Element in Array
   - **Context**: Find the kth largest element in unsorted array
   - **Example**: [3,2,1,5,6,4], k=2 → return 5

```
import heapq

def findKthLargest(nums, k):
    return heapq.nlargest(k, nums)[-1]
```
# Or using min heap of size k
```
def findKthLargest(nums, k):
    heap = nums[:k]
    heapq.heapify(heap)
    
    for num in nums[k:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)
    
    return heap[0]
```

2. Merge K Sorted Lists
   - **Context**: Merge k sorted linked lists into one sorted list
   - **Challenge**: Efficiently compare heads of all lists

```
import heapq

def mergeKLists(lists):
    heap = []
    
    # Add first node of each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst.val, i, lst))
    
    dummy = ListNode(0)
    current = dummy
    
    while heap:
        val, i, node = heapq.heappop(heap)
        current.next = node
        current = current.next
        
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    
    return dummy.next
```

3. Top K Frequent Elements
   - **Context**: Find k most frequent elements in array
   - **Example**: [1,1,1,2,2,3], k=2 → return [1,2]

```
import heapq
from collections import Counter

def topKFrequent(nums, k):
    count = Counter(nums)
    return heapq.nlargest(k, count.keys(), key=count.get)
```
# Or using min heap
```
def topKFrequent(nums, k):
    count = Counter(nums)
    heap = []
    
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
    
    return [num for freq, num in heap]
```

These problems test core heap concepts: priority queues, efficient min/max finding, and maintaining top-k elements.

