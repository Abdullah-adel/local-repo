## first challenge, Bad runtime :) 

#### is this the better solution for leet code challenge Two sum: class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for num1 in range(len(nums) - 1):
            for num2 in range(num1+1, len(nums)):
                if nums[num1] + nums[num2] == target:
                    return [num1, num2]

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

