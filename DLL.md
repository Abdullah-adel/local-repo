### DLL: Palindrome Checker ( ** Interview Question)
>Write a method to determine whether a given doubly linked list reads the same forwards and backwards.
>For example, if the list contains the values [1, 2, 3, 2, 1], then the method should return True, since the list is a palindrome.

```
    def is_palindrome(self):
        #if self.length
        if self.length <= 1:
            return True
        
        forward = self.head
        backword = self.tail
        
        for _ in range((self.length) // 2 ):
            if forward.value == backword.value:
                forward = forward.next
                backword = backword.prev
            else:
                return False
        return True

```

### DLL_Leet reverse challenge:
>Create a new method called reverse that reverses the order of the nodes in the list, i.e., the first node becomes the last node, the second node becomes the second-to-last node, and so on.
>To do this, you'll need to traverse the list and change the direction of the pointers between the nodes so that they point in the opposite direction.
>Do not change the value of any of the nodes.
>Once you've done this for all nodes, you'll also need to update the head and tail pointers to reflect the new order of the nodes.


 Solution #1:
```
    def reverse(self):
            if not self.head or not self.head.next:
                return
            
            current = self.head
            temp = None
            
            while current:
                temp = current.prev
                current.prev = current.next
                current.next = temp
                current = current.prev
            
            temp = self.head
            self.head = self.tail
            self.tail = temp
```

Solution #2
```
    def reverse(self):
        temp = self.head
        while temp is not None:
            # swap the prev and next pointers of node points to
            temp.prev, temp.next = temp.next, temp.prev
            
            # move to the next node
            temp = temp.prev
            
        # swap the head and tail pointers
        self.head, self.tail = self.tail, self.head
```



### DLL_Leet Partition challenge:
> Write a method called partition_list(self, x) that rearranges the nodes in a doubly linked list 
so that all nodes with a value less than a given number x come before all nodes with a value greater than or equal to x.


Code with inline comments:


```
def partition_list(self, x):
    # ------------------------------------------------
    # If the list is empty, return immediately
    # Nothing to partition
    # ------------------------------------------------
    if not self.head:
        return None
 
    # ------------------------------------------------
    # Create two dummy nodes to serve as the starting
    # points for the two new partitions
    # dummy1 → nodes with values < x
    # dummy2 → nodes with values ≥ x
    # ------------------------------------------------
    dummy1 = Node(0)
    dummy2 = Node(0)
 
    # ------------------------------------------------
    # prev1 tracks the end of the < x partition
    # prev2 tracks the end of the ≥ x partition
    # ------------------------------------------------
    prev1 = dummy1
    prev2 = dummy2
 
    # Start at the head of the original list
    current = self.head
 
    # ------------------------------------------------
    # Traverse the original list and divide nodes
    # into two separate partitions based on their value
    # ------------------------------------------------
    while current:
        if current.value < x:
            # ----------------------------------------
            # Append current node to the < x list
            # Update pointers to maintain .next/.prev
            # ----------------------------------------
            prev1.next = current
            current.prev = prev1
            prev1 = current
        else:
            # ----------------------------------------
            # Append current node to the ≥ x list
            # Update pointers to maintain .next/.prev
            # ----------------------------------------
            prev2.next = current
            current.prev = prev2
            prev2 = current
 
        # Move to the next node in the original list
        current = current.next
 
    # ------------------------------------------------
    # Terminate the ≥ x list to prevent cycle or
    # trailing data from previous .next values
    # ------------------------------------------------
    prev2.next = None
 
    # ------------------------------------------------
    # Connect the two partitions:
    # Link the end of the < x list to the beginning
    # of the ≥ x list
    # ------------------------------------------------
    prev1.next = dummy2.next
 
    # ------------------------------------------------
    # If the ≥ x list has at least one node,
    # update its .prev to point to the < x list
    # ------------------------------------------------
    if dummy2.next:
        dummy2.next.prev = prev1
 
    # ------------------------------------------------
    # Update the head of the list to the start of
    # the < x partition (after dummy1)
    # ------------------------------------------------
    self.head = dummy1.next
 
    # ------------------------------------------------
    # Ensure the new head has no previous pointer
    # (important for DLL structure)
    # ------------------------------------------------
    self.head.prev = None
```

### DLL_Leet reverse_between challenge:

🧠 Explanation of the Code

We want to reverse a sublist within a doubly linked list from start_index to end_index.

We use a dummy node to simplify edge cases like modifying the head of the list.

We walk forward from the dummy to find the node before the reversal segment (prev).

current points to the start of the reversal segment.

In each iteration, we:

Remove current.next and re-insert it immediately after prev

This results in an in-place reversal of the sublist

After reversing, we reset the .head in case it changed due to head being part of the segment.

We ensure that self.head.prev is reset to None.





🔧 Code with Inline Comments


```
def reverse_between(self, start_index, end_index):
    # ---------------------------------------------
    # Reverses the portion of the list between the
    # given start_index and end_index in-place.
    # Assumes 0-based indexing.
    # ---------------------------------------------
 
    # If the list has 0 or 1 nodes, or no change needed
    if self.length <= 1 or start_index == end_index:
        return
 
    # Create a dummy node before head to simplify edge cases
    dummy = Node(0)
    dummy.next = self.head
    self.head.prev = dummy
 
    # Traverse to the node just before the start_index
    prev = dummy
    for _ in range(start_index):
        prev = prev.next
 
    # current points to the first node in the segment to reverse
    current = prev.next
 
    # Reverse the segment using node splicing
    for _ in range(end_index - start_index):
        node_to_move = current.next
 
        # Detach node_to_move from its current position
        current.next = node_to_move.next
        if node_to_move.next:
            node_to_move.next.prev = current
 
        # Insert node_to_move right after prev
        node_to_move.next = prev.next
        prev.next.prev = node_to_move
        prev.next = node_to_move
        node_to_move.prev = prev
 
    # Update head pointer in case it was changed
    self.head = dummy.next
    self.head.prev = None
```
