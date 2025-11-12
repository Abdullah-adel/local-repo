
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
