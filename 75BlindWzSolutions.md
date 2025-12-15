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
