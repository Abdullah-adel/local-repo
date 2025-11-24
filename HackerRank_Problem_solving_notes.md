
-----------------------------------
#### 2D arr - diagonalDifference
-----------------------------------

write a python code to solve: Given a square matrix, calculate the absolute difference between the sums of its diagonals.

> python
```
def diagonalDifference(arr):
    n = len(arr)
    primary = sum(arr[i][i] for i in range(n))
    secondary = sum(arr[i][n-1-i] for i in range(n))
    return abs(primary - secondary)
```

How it works:
- Primary diagonal: elements where row index = column index arr[i][i]
- Secondary diagonal: elements where row + column = n-1, so arr[i][n-1-i]
- Return absolute difference between the two sums

-----------------------------------
#### Plus Minus
-----------------------------------

 write a python code that Given an array of integers, calculate the ratios of its elements that are positive, negative , zeros and . Print the decimal value of each fraction on a new line with 6 places after the decimal.

--> two version mine **17 lines** :D Vs a tenured version of **7 lines**
### my code: 
```
def plusMinus(arr):
    total = len(arr)
    positive_count = 0
    negative_count = 0
    zeros_count = 0
    for i in range(total):
        if arr[i] > 0:
            positive_count += 1
        elif arr[i] < 0:
            negative_count += 1
        else:
            zeros_count += 1
    frac1 = positive_count/total
    fract2 = negative_count/total
    fract3 = zeros_count/total
    print(f"{frac1:.6f}")
    print(f"{fract2:.6f}")
    print(f"{fract3:.6f}")
```    

### More consice and comperhensive one
```
def plusMinus(arr):
    n = len(arr)
    positive = sum(1 for x in arr if x > 0)
    negative = sum(1 for x in arr if x < 0)
    zero = sum(1 for x in arr if x == 0)
    
    print(f"{positive/n:.6f}")
    print(f"{negative/n:.6f}")
    print(f"{zero/n:.6f}")
```

### miniMaxSum challenge

python
```
def miniMaxSum(arr):
    min_num = arr[0]  # Initialize with first element
    max_num = arr[0]  # Initialize with first element
    total_sum = 0
    
    for num in arr:
        if num > max_num:
            max_num = num
        if num < min_num:  # Separate condition for minimum
            min_num = num
        total_sum += num
    
    min_sum = total_sum - max_num
    max_sum = total_sum - min_num
    print(min_sum, max_sum)
```

Even simpler approach:
python
```
def miniMaxSum(arr):
    total_sum = sum(arr)
    min_sum = total_sum - max(arr)
    max_sum = total_sum - min(arr)
    print(min_sum, max_sum)
```

### Birthday Cake Candles 
my solution:
------------
```
def birthdayCakeCandles(candles):
    tallest = max(candles)
    counter = 0
    for candel in candles:
        if candel == tallest:
            counter += 1
    return counter
```

my simpler code:
----------------
 ```
def birthdayCakeCandles(candles):
    max_height = max(candles)
    counter = sum( 1 for candle in candles if candle == max_height)
    return counter
```

Even simpler:
--------------
python
```
def birthdayCakeCandles(candles):
    return candles.count(max(candles))
```
============
 #### Number Line Jumps 
They will land on the same spot only if the difference in starting positions is divisible by the difference in speeds, and the faster one starts behind the slower one.[1][2]

## Problem idea

Two kangaroos start at positions $$x1$$ and $$x2$$ and jump forward with speeds $$v1$$ and $$v2$$.[3]
After $$n$$ jumps, their positions are:

- First: $$x1 + n \cdot v1$$  
- Second: $$x2 + n \cdot v2$$[4]

They meet if there is some non‑negative integer $$n$$ with:
$$
x1 + n \cdot v1 = x2 + n \cdot v2
$$
Rearrange:
$$
n = \frac{x2 - x1}{v1 - v2}
$$
[4]

For them to meet:
- $$v1 \ne v2$$ (otherwise distances never change).[2]
- $$x2 - x1$$ is divisible by $$v1 - v2$$ (so $$n$$ is an integer).[1]
- $$n \ge 0$$, meaning the one behind must be faster.[2]

***

## Simple math-based Python solution

```python
def kangaroo(x1, v1, x2, v2):
    # If speeds are equal, they only meet if they start together
    if v1 == v2:
        return "YES" if x1 == x2 else "NO"
    
    # Compute n = (x2 - x1) / (v1 - v2)
    diff_pos = x2 - x1
    diff_vel = v1 - v2
    
    # They must move towards each other and meet at an integer step
    if diff_pos % diff_vel == 0 and diff_pos / diff_vel >= 0:
        return "YES"
    else:
        return "NO"
```

Explanation in words:

- If speeds same:  
  - same start → always together → "YES"  
  - different start → distance never changes → "NO".[2]
- Otherwise, check if the division gives a whole, non‑negative number of jumps.

***

## Simple simulation solution (loop)

This is easier to think about, but slower (fine for HackerRank constraints).[5]

```python
def kangaroo(x1, v1, x2, v2):
    # Limit jumps so we don't loop forever
    for _ in range(10000):
        if x1 == x2:
            return "YES"
        x1 += v1
        x2 += v2
    return "NO"
```

Explanation:

- Move both forward one jump at a time.  
- If positions ever match → "YES"; if not after many jumps → assume "NO".[5]

***

## Very short Python solution

This is just the math version written compactly.

```python
def kangaroo(x1, v1, x2, v2):
    return "YES" if (v1 != v2 and (x2 - x1) % (v1 - v2) == 0 and (x2 - x1) / (v1 - v2) >= 0) or (x1 == x2 and v1 == v2) else "NO"
```

You can use the math-based version in contests and the loop version to understand the behaviour step by step.

#### "Between Two Sets"  challenge
Here's a simple beginner-friendly solution for the "Between Two Sets" problem without using extra functions or the functools module:

### Explanation

- Find the maximum number in array `a` (call it `max_a`).
- Find the minimum number in array `b` (call it `min_b`).
- Check every number `x` from `max_a` up to `min_b`.
- For each `x`, check if:
  1. All elements in `a` divide `x` evenly.
  2. `x` divides all elements in `b` evenly.
- Count how many such numbers satisfy both conditions.

### Simple Code Example in Python

```python
def getTotalX(a, b):
    max_a = max(a)
    min_b = min(b)
    count = 0

    for x in range(max_a, min_b + 1):
        # Check if all elements in a divide x
        divisible_by_a = True
        for num in a:
            if x % num != 0:
                divisible_by_a = False
                break
        
        if not divisible_by_a:
            continue
        
        # Check if x divides all elements in b
        divides_b = True
        for num in b:
            if num % x != 0:
                divides_b = False
                break
        
        if divides_b:
            count += 1
    
    return count

# Example usage:
a = [2, 4]
b = [16, 32, 96]
print(getTotalX(a, b))  # Output: 3
```

### How it works

- Start at the biggest number in `a` since smaller numbers can't satisfy the first condition if bigger numbers in `a` don't.
- Go up to the smallest number in `b` because numbers bigger than that can't divide all elements in `b`.
- For each candidate number, check both conditions with loops and count how many pass.

This approach is straightforward and good for beginners learning loops and conditionals.[1]

[1](https://studyalgorithms.com/array/hackerrank-between-two-sets/)

