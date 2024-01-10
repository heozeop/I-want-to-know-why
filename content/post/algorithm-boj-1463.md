---
title: "Algorithm Boj 1463"
date: 2023-11-14T18:29:54+09:00
draft: true

categories:
- algorithm
---

### 문제
- [1로 만들기](https://www.acmicpc.net/problem/1463)

### 풀이
- 생각나는 방법
    - n짜리 넣어 두고 1가는 횟수 찾기
    - 1에 먼저 도착하면 그게 최소
- 시간 복잡도
    - O(n)
- 코드
```python
n = int(input())

INF = 1e9

arr = [INF] * (n + 1)

arr[n] = 0

for i in range(n, 0, -1):
    if i % 3 == 0:
        arr[int(i/3)] = min(arr[int(i/3)], arr[i] + 1)
    if i % 2 == 0:
        arr[int(i/2)] = min(arr[int(i/2)], arr[i] + 1)
    arr[i-1] = min(arr[i-1], arr[i] + 1)
    
print(arr[1])
```

- 다른 방법
    - dfs
