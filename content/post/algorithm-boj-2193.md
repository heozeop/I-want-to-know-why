---
title: "Algorithm Boj 2193"
date: 2023-11-17T21:34:36+09:00
draft: true

categories:
- algorithm
---

### 문제
- [이친수](https://www.acmicpc.net/problem/2193)

### 풀이
- 핵심
    - dp[i][k] => i 자리 끝이 k인 이친수
        - dp[i][0] = dp[i-1][1] + dp[i -1][0]
        - dp[i][1] = dp[i-1][0]
        - dp[1][0] = 0
        - dp[1][1] = 1
- 구체적인 코드
    - n받는다.
    - dp n, 2짜리 2차원 arr만든다
    - dp[1][0] = 0, dp[1][1] = 1
    - for i in 2 ~ n:
        - dp[i][0] = dp[i-1][1] + dp[i-1][0]
        - dp[i][1] = dp[i-1][0]
    - print(dp[n][0] + dp[n][1])
- 코드
```python
n = int(input())

dp = [[0] * 2 for _ in range(n + 1)]
dp[1][0] = 0
dp[1][1] = 1

for i in range(2, n + 1):
    dp[i][0] = dp[i-1][0] + dp[i-1][1]
    dp[i][1] = dp[i-1][0]
    
print(dp[n][0] + dp[n][1])
```
