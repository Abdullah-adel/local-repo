Here's a more technical breakdown of Big O and DSA:

Big O Notation Common Types:
1. O(1) - Constant Time
- Example: Accessing array element by index
```python
array[0] # instant access regardless of array size
```
- Big O is always about measuring the worst case
- Drop constant such in O(2n) = o(n)
- Drop non-dominant such in: O(n -1) = O(n)

#### Python lists store their references in contiguous memory locations, but the actual objects may not be contiguous.

How Python lists work:
• The list structure itself is an array of pointers stored contiguously
• Each pointer points to the actual object somewhere in memory
• The objects themselves can be scattered anywhere

#### No, linked lists are NOT stored in contiguous memory locations.

How linked lists work:
• Each node can be stored anywhere in memory
• Nodes are connected through pointers/references
• Memory addresses are scattered and random


LL --> is different than a list where it is located in contiguos memory locations and does not have indexes
- the Node is (value and pointer to next value), you construct a node with dict. node = {'value': 7, 'next': {'value' = 4 , 'next' = None}}
- **Reverse LL** is very common interview question

### Common interview Q:
To detect the loop -> **Floyd's cycle-finding algorithm** (also known as the "tortoise and the hare" algorithm)
This algorithm uses two pointers: a slow pointer and a fast pointer. The slow pointer moves one step at a time, 
while the fast pointer moves two steps at a time. If there is a loop in the linked list, the two pointers will eventually meet at some point. 
If there is no loop, the fast pointer will reach the end of the list.

==Code:
    def has_loop(self):
        slow = self.head
        fast = self.head
        while fast is not None and fast.next is not None:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True
            
        return False


##### LL: Find Kth Node From End ( ** Interview Question)
  The find_kth_from_end function should follow these requirements:
  The function should utilize two pointers, slow and fast, initialized to the head of the linked list.
  The fast pointer should move k nodes ahead in the list.
  If the fast pointer becomes None before moving k nodes, the function should return None, as the list is shorter than k nodes.
  The slow and fast pointers should then move forward in the list at the same time until the fast pointer reaches the end of the list.
  The function should return the slow pointer, which will be at the k-th position from the end of the list.

=== Code:
def find_kth_from_end(ll, k):       
    slow = fast = ll.head
    for _ in range(k):
        if fast is None:
            return None
        fast = fast.next
    while fast:
        slow = slow.next
        fast = fast.next
    return slow    

== LL leet: Remove duplicates:

You can implement the remove_duplicates() method in two different ways:
- Using a Set - This approach will have a time complexity of O(n), where n is the number of nodes in the linked list. 
You are allowed to use the provided Set data structure in your implementation.
- Without using a Set - This approach will have a time complexity of O(n^2), where n is the number of nodes in the linked list. 
You are not allowed to use any additional data structures for this implementation.
----------------------
Solution using nested loops:
----------------------
    def remove_duplicates(self):
        current = self.head
        while current:
            # Start checking from the next node
            runner = current
            while runner.next:
                # If the next node's value matches current's value, skip it
                if runner.next.value == current.value:
                    runner.next = runner.next.next
                    self.length -= 1
                else:
                    runner = runner.next
            current = current.next


----------------------
Solution using a set:
----------------------
    def remove_duplicates(self):
            values = set()
            previous = None
            current = self.head
            while current:
                if current.value in values:
                    previous.next = current.next
                    self.length -= 1
                else:
                    values.add(current.value)
                    previous = current
                current = current.next

----------------------
binary_to_decimal
----------------------
The binary_to_decimal method should start from the head of the linked list and use each node's value to calculate the corresponding decimal number. The formula to convert a binary number to decimal is as follows:

To put it in simple terms, each digit of the binary number is multiplied by 2 raised to the power equivalent to the position of the digit, counting from right to left starting from 0, and all the results are summed together to get the decimal number.
--> For every node we encounter, we:
  - Multiply the current num by 2. This shifts our binary number left, preparing it for the next bit. 
    It's analogous to multiplying by 10 in decimal arithmetic.
  - Add the value of the current node (0 or 1) to num.
==Code:
    def binary_to_decimal(self):
        num = 0
        current = self.head
        while current:
            num = num * 2 + current.value
            current = current.next
        return num

----------------------
partition_list
----------------------
Implement the partition_list member function for the LinkedList class, which partitions the list such that all nodes with values less than x come before nodes with values greater than or equal to x.
Note:  This linked list class does NOT have a tail which will make this method easier to implement.
The original relative order of the nodes should be preserved.

==Code:
    def partition_list(self, x):
        if not self.head:
            return None
    
        dummy1 = Node(0)
        dummy2 = Node(0)
        prev1 = dummy1
        prev2 = dummy2
        current = self.head
    
        while current:
            if current.value < x:
                prev1.next = current
                prev1 = current
            else:
                prev2.next = current
                prev2 = current
            current = current.next
    
        prev1.next = dummy2.next
        prev2.next = None
    
        self.head = dummy1.next


----------------------
LL: Reverse Between ( ** Interview Question)
----------------------
Welcome to what many consider the pinnacle of Linked List challenges 
This exercise is not just a test of your coding skills, but also a measure of your problem-solving ability and understanding of complex data structures.

Here's how you can tackle it:

Visualize the Problem: Before diving into coding, grab a pen and paper. Sketch out the linked list and visualize each step of the process. This approach isn't just for beginners; it's exactly how seasoned developers plan their attack on complex problems.

Seek Understanding Over Speed: Take your time to really grasp each part of the problem. The goal here is deep understanding, not just a quick solution. If you find yourself stuck, that's a part of the learning process.

It's Okay to Take a Break: Feel free to step away from this challenge and return later. This course is designed to equip you with a growing set of skills, and sometimes, a bit of distance can bring new insights.

Approach Like a Pro: Remember, facing a challenging problem is what being a professional developer is all about. Use this exercise to think, plan, and code like a pro.

==Code:
    def reverse_between(self, start_index, end_index):
        if self.length <= 0:
            return
        
        dum_node =Node(0)
        dum_node.next = self.head
        prev = dum_node
        
        for i in range(start_index):
            prev = prev.next
        
        curr_node = prev.next
        
        for i in range(end_index - start_index):
            node_to_move = curr_node.next
            curr_node.next = node_to_move.next
            node_to_move.next = prev.next
            prev.next = node_to_move
    
--> Notice:
You CANNOT:

Move
node_to_move = curr_node.next
from first position because other operations depend on this value

-> Put
curr_node.next = node_to_move.next
-> after
node_to_move.next = prev.next

because you'll lose the reference
The reason is:
First line must store the reference to the node we're moving
We need to maintain proper references before changing them
Breaking these dependencies would cause loss of node references and incorrect linking


----------------------
LL: Swap Nodes in Pairs
----------------------

Code:
        while first and first.next:
            second = first.next
            
            # Swap next pointers
            previous.next = second
            first.next = second.next
            second.next = first
        
            # Move pointers
            previous = first
            first = first.next
            
        self.head = dummy.next







Basic Operations:
• Reverse a linked list (iterative and recursive)
• Detect cycle in linked list (Floyd's algorithm) ... Floyd's cycle-finding algorithm (also known as the "tortoise and the hare" algorithm)
• Find middle of linked list
• Remove nth node from end
• Merge two sorted linked lists

Advanced Problems:
• Remove duplicates from sorted/unsorted list
• Add two numbers represented as linked lists
• Intersection of two linked lists
• Palindrome linked list check
• Rotate linked list by k positions

Key Patterns to Master:
• Two pointers (fast/slow)
• Dummy head node technique
• In-place reversal
• Stack for comparison problems

Most Frequent:
1. Reverse Linked List - appears in 80%+ of interviews
2. Cycle Detection - classic Floyd's tortoise and hare
3. Merge Two Lists - tests basic pointer manipulation
4. Remove Nth from End - two-pointer technique

Pro Tips:
• Always handle edge cases (null, single node)
• Draw diagrams during explanation
• Consider both iterative and recursive solutions
• Practice with and without dummy nodes

2. O(log n) - Logarithmic Time
- Example: Binary Search
```python
def binary_search(arr, target):
    left, right = 0, len(arr)-1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target: return mid
        if arr[mid] < target: left = mid + 1
        else: right = mid - 1
```

3. O(n) - Linear Time
- Example: Linear Search
```python
def linear_search(arr, target):
    for i in range(len(arr)):
        if arr[i] == target: return i
```

4. O(n²) - Quadratic Time
- Example: Bubble Sort
```python
def bubble_sort(arr):
    for i in range(len(arr)):
        for j in range(len(arr)-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
```

Important Data Structures:

1. Arrays/Lists
- Good: Fast access O(1)
- Bad: Slow insertion/deletion O(n)
```python
arr = [1,2,3] # Access: arr[0] is O(1)
```

2. Hash Tables/Dictionaries
- Good: Fast access/insertion O(1)
- Bad: More memory usage
```python
dict = {'key': 'value'} # Access: dict['key'] is O(1)
```

3. Linked Lists
- Good: Fast insertion/deletion O(1)
- Bad: Slow access O(n)
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
```

4. Trees (Binary Search Tree)
- Good: Balanced search O(log n)
- Bad: Can become unbalanced
```python
class TreeNode:
    def __init__(self, data):
        self.data = data
        self.left = None
        self.right = None
```

Real-World Applications:

1. Database Indexing
- Uses B-trees for efficient searching
- Turns O(n) operations into O(log n)

2. Social Media Feed
- Uses heap data structure for prioritizing content
- Efficient insertion O(log n)

3. GPS/Maps
- Uses graph algorithms (Dijkstra's) for shortest path
- Complex routing becomes manageable

4. Browser History
- Uses stack data structure
- O(1) for back/forward operations

Common Algorithm Paradigms:

1. Divide and Conquer
- Example: Merge Sort O(n log n)
- Splits problem into smaller sub-problems

2. Dynamic Programming
- Example: Fibonacci with memoization
- Stores sub-results to avoid recalculation

3. Greedy Algorithms
- Example: Dijkstra's shortest path
- Makes locally optimal choices

Memory Complexity:
- Space complexity is as important as time
- Example: Recursive vs Iterative solutions
- Recursion might be O(n) space while iteration O(1)

This knowledge is crucial for:
1. System Design
2. Performance Optimization
3. Technical Interviews
4. Scaling Applications

Understanding these concepts helps write efficient code that can handle large-scale applications and data processing.



#### Dynamic Programming:
* Overlapping subproblems: it means repeating subproblems .. storing the answers of subproblems is called `Memoization`.
Dynamic programming requirements:
 - Repeating subporblems "Overlapping subproblems"
 - Optimized structure.

