
> Sorting a string in Python:

Method 1: Using sorted() (most common)
python
s = "hello"
sorted_string = ''.join(sorted(s))  # "ehllo"


Method 2: Convert to list, sort, join back
python
s = "hello"
char_list = list(s)
char_list.sort()
sorted_string = ''.join(char_list)  # "ehllo"


Method 3: Reverse order
python
s = "hello"
sorted_string = ''.join(sorted(s, reverse=True))  # "ollhe"


Most common and simple:
python
sorted_string = ''.join(sorted(original_string))


Key points:
- sorted() returns a list of characters
- ''.join() combines them back into a string
- Sorts alphabetically by default
- Case-sensitive (uppercase comes before lowercase)


## first challenge, Bad runtime :) 

#### is this the better solution for leet code challenge Two sum: class Solution:
   ```
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for num1 in range(len(nums) - 1):
            for num2 in range(num1+1, len(nums)):
                if nums[num1] + nums[num2] == target:
                    return [num1, num2]
```
> Your solution works but is not optimal. It's O(n²) time complexity.

Better optimized solution using hash map: O(n)
python
```
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_map = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in num_map:
                return [num_map[complement], i]
            num_map[num] = i
```

Why it's better:
- **Time:** O(n) vs O(n²) 
- **Space:** O(n) for hash map vs O(1)
- **Speed:** Single pass vs nested loops

How it works:
1. For each number, calculate what we need to reach target
2. Check if we've seen that complement before
3. If yes, return both indices
4. If no, store current number and its index

Example: nums=[2,7,11,15], target=9
- i=0, num=2: complement=7, not seen → store {2:0}
- i=1, num=7: complement=2, found at index 0 → return [0,1]

The hash map approach is the standard optimal solution for Two Sum.

> how to solve isPalindrome for the above

> isPalindrome for strings (ignoring case and non-alphanumeric):

Method 1: Clean then compare
`
def isPalindrome(s: str) -> bool:
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    return cleaned == cleaned[::-1]
`

Method 2: Two pointers (more efficient)
```
def isPalindrome(s: str) -> bool:
    left, right = 0, len(s) - 1
    
    while left < right:
        # Skip non-alphanumeric from left
        while left < right and not s[left].isalnum():
            left += 1
        
        # Skip non-alphanumeric from right
        while left < right and not s[right].isalnum():
            right -= 1
        
        # Compare characters
        if s[left].lower() != s[right].lower():
            return False
        
        left += 1
        right -= 1
    
    return True
```

Most simple and readable:
```
def isPalindrome(s: str) -> bool:
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    return cleaned == cleaned[::-1]
```

Example:
```
isPalindrome("A man, a plan, a canal: Panama")  # True
isPalindrome("race a car")                       # False
```

### The first method is cleaner and easier to understand, while the second is more memory efficient.

