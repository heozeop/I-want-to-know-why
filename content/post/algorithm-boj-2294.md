---
title: "Algorithm Boj 2294"
date: 2023-11-17T21:07:04+09:00
draft: true

categories:
- algorithm
---

### 문제
- [동전2](https://www.acmicpc.net/problem/2294)

### 풀이
- 핵심
    - 종류로 DP를 만든다.
    - DP[K]는 K를 만들 수 있는 최소 동전 개수다.
- 구체적인 로직
    - n,k 받기
    - arr에 array 받기
    - for i ~ n
        - dp[i][k] = min(dp[i-1][k], dp[k - arr[i]], k % i == 0 ? k / i : INF)
    - print dp[n][k]
- 주의
    - 문제를 제대로 읽지 않아서 구할 수 없는 경우를 처리하지 못했다.
    - 코드로 옮기기 전에 조건을 다시한번 확인하자.
- 코드
```python
n,k = list(map(int, input().split()))

arr = [0,]

for _ in range(n):
    arr.append(int(input()))

INF = 1e9
dp = [[INF] * (k + 1) for _ in range(n + 1)]
for i in range(1, n + 1):
    for j in range(1, k + 1):
        dp[i][j] = dp[i-1][j]
        if j % arr[i] == 0:
            dp[i][j] = min(dp[i][j], int(j / arr[i]))
        
        if j - arr[i] >= 0:
            dp[i][j] = min(dp[i][j], dp[i][j - arr[i]] + 1)
if dp[n][k] == INF:
    print(-1)
else:
    print(dp[n][k])
```
    

