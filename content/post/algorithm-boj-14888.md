---
title: "Boj 14888"
date: 2023-10-06T19:53:38+09:00
draft: false

categories:
- algorithm
---

# 주요 로직
- 계속 계산하면서 큰값과 작은 값으로 비교한다.
- 전체 다 돌아야 하며, O(4n)이다.
- -10억 ~ 10억이므로 int 범위에서 된다.

## 예상하지 못한 것
- 음수가 나오는 경우, 나눗셈 연산을 할때 몫을 잘 구해야한다.
- 음수가 -10억까지 되는데 범위를 잘못 찍었다.

## code
 ```python3
 n = int(input())
nums = list(map(int, input().split()))
ops = list(map(int,input().split()))

maxVal = -1000000001
minVal = 1000000001

def calc(calcVal, operation, index):
    if operation == 0:
        return calcVal + nums[index]
    elif operation == 1:
        return calcVal - nums[index]
    elif operation == 2:
        return calcVal * nums[index]
    elif operation == 3:
        if calcVal < 0:
            return -1 * ((-1 * calcVal) // nums[index])
        return calcVal // nums[index]

def dfs(prevIndex, calcVal):
    global maxVal
    global minVal
    
    if prevIndex == n - 1:
        maxVal = max(maxVal, calcVal)
        minVal = min(minVal, calcVal)
        return
    
    for i in range(4):
        if ops[i] == 0:
            continue
        ops[i] -= 1
        dfs(prevIndex + 1, calc(calcVal, i, prevIndex + 1))
        ops[i] += 1

dfs(0, nums[0])
print(maxVal)
print(minVal)
 ```
 
