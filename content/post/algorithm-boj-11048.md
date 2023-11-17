---
title: "Algorithm Boj 11048"
date: 2023-11-17T21:52:49+09:00
draft: false

categories:
- algorithm
---

### 문제
- [이동하기](https://www.acmicpc.net/problem/11048)

### 풀이
- 핵심
    - dp[i][j] => i,j에서 먹을 수 있는 최대 값
        - dp[i][j] = max(dp[i][j-1], dp[i-1][j-1], dp[i-1][j]) + arr[i][j]
- 구체적인 풀이
    - n,m 받기
    - arr로 int전환해서 map받기
    - for i in n
        - for j in m
            - dp ~
    - print(dp[n - 1][m - 1])
- 코드
```python
n,m = list(map(int, input().split()))

arr = []
for _ in range(n):
    arr.append(list(map(int, input().split())))
    
dp = [[0] * (m + 1) for _ in range(n + 1)]

for i in range(1, n+ 1):
    for j in range(1, m+1):
        dp[i][j] = max(dp[i][j-1], dp[i-1][j-1], dp[i-1][j]) + arr[i-1][j-1]
print(dp[n][m])
```
    

