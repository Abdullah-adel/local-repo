
-----------------------------------
#### 2D arr - diagonalDifference
-----------------------------------

write a python code to solve: Given a square matrix, calculate the absolute difference between the sums of its diagonals.

> python
def diagonalDifference(arr):
    n = len(arr)
    primary = sum(arr[i][i] for i in range(n))
    secondary = sum(arr[i][n-1-i] for i in range(n))
    return abs(primary - secondary)


How it works:
• Primary diagonal: elements where row index = column index arr[i][i]
• Secondary diagonal: elements where row + column = n-1, so arr[i][n-1-i]
• Return absolute difference between the two sums
