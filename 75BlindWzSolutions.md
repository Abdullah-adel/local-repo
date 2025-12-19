###  Best Time to Buy and Sell Stock
You are given an array prices where prices[i] is the price of a given stock on the ith day.

You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock.

```
    def maxProfit(self, prices: List[int]) -> int:
        min_price = prices[0]
        max_profit = 0

        for price in prices:
            max_profit = max(max_profit, price - min_price)
            min_price = min(price, min_price)
        return max_profit
```

Better performance one:
[Solution explanation](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/solutions/4868897/most-optimized-kadanes-algorithm-java-c-2yt85/)
```
class Solution:
    def maxProfit(self, prices):
        buy = prices[0]
        profit = 0
        for i in range(1, len(prices)):
            if prices[i] < buy:
                buy = prices[i]
            elif prices[i] - buy > profit:
                profit = prices[i] - buy
        return profit
```
### Product of Array Except Self:
You can solve “Product of Array Except Self” in O(n) using prefix and suffix products, without division. The main idea is: for each index i, multiply “product of all elements to the left of i” with “product of all elements to the right of i”.
```
def productExceptSelf(nums):
    n = len(nums)
    ans = [1] * n

    # Left pass: ans[i] = product of all elements to the left of i
    left = 1
    for i in range(n):
        ans[i] = left
        left *= nums[i]

    # Right pass: multiply by product of all elements to the right of i
    right = 1
    for i in range(n - 1, -1, -1):
        ans[i] *= right
        right *= nums[i]

    return ans
```

### leetcode :  Maximum Subarray

```
def maxSubArray(self, nums: List[int]) -> int:
    max_sum = nums[0]
    current_sum = nums[0]
    
    for i in range(1, len(nums)):
        current_sum = max(nums[i], current_sum + nums[i])
        max_sum = max(max_sum, current_sum)
    
    return max_sum
```

## How it works:

Key insight: At each position, decide whether to:
1. Start a new subarray from current element
2. Extend the existing subarray
```
Example: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

i=1: current_sum = max(1, -2+1) = 1,     max_sum = 1
i=2: current_sum = max(-3, 1-3) = -2,    max_sum = 1  
i=3: current_sum = max(4, -2+4) = 4,     max_sum = 4
i=4: current_sum = max(-1, 4-1) = 3,     max_sum = 4
i=5: current_sum = max(2, 3+2) = 5,      max_sum = 5
i=6: current_sum = max(1, 5+1) = 6,      max_sum = 6
i=7: current_sum = max(-5, 6-5) = 1,     max_sum = 6
i=8: current_sum = max(4, 1+4) = 5,      max_sum = 6


Result: 6 (subarray [4, -1, 2, 1])
```
## This is Kadane's algorithm - O(n) time, O(1) space.
-------------

### Maximum Product Subarray

```
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        max_product = min_product = result = nums[0]

        for i in range(1, len(nums)):
            if nums[i] < 0 :
                min_product, max_product = max_product , min_product
            min_product = min(nums[i], min_product * nums[i])
            max_product = max(nums[i], max_product * nums[i])
            result = max(result, max_product)

        return result
```

> Let me trace through nums = [2, 3, -2, 4] step by step:
```
## Initial state:
max_prod = 2, min_prod = 2, result = 2


## Step 1: nums[1] = 3
nums[i] = 3 (positive, no swap needed)
max_prod = max(3, 2 * 3) = max(3, 6) = 6
min_prod = min(3, 2 * 3) = min(3, 6) = 3  
result = max(2, 6) = 6


## Step 2: nums[2] = -2
nums[i] = -2 (negative, so swap max_prod and min_prod)
Before swap: max_prod = 6, min_prod = 3
After swap:  max_prod = 3, min_prod = 6

max_prod = max(-2, 3 * -2) = max(-2, -6) = -2
min_prod = min(-2, 6 * -2) = min(-2, -12) = -12
result = max(6, -2) = 6


## Step 3: nums[3] = 4  
nums[i] = 4 (positive, no swap needed)
max_prod = max(4, -2 * 4) = max(4, -8) = 4
min_prod = min(4, -12 * 4) = min(4, -48) = -48
result = max(6, 4) = 6


## Final result: 6
```
Why we swap on negative numbers:
- Before -2: we had max_prod=6 (from 2×3)
- After swap: min_prod=6, so when we multiply by -2, we get 6×(-2)=-12
- This -12 becomes our new minimum, which could become maximum if we hit another negative

The maximum subarray product is 6 (from subarray [2,3]).

