---
title: "Algorithm Boj 16437"
date: 2023-11-08T08:27:40+09:00
draft: false

categories:
- algorithm
---

### 문제
- [양 구출 작전](https://www.acmicpc.net/problem/16437)

### 풀이
- 핵심
    - 양있는 섬에서 1번 섬까지 가기
    - 가는게 독립시행
- 자세히
    - 맵으로 트리 만들기
    - 트리 순회 해서 1번 섬으로 도착하는 것 하기
    - 양 있는데는 한번씩 돌기
- 변수
    - island_map
    - sheep_land_list
- 시간 복잡도
    - O(n^2)
        - m_i: i번째 길 수
        - n: 1번까지 path 길이
        - 수열의 합
---
- 간과 한것
    1. 사이클, 도달할 수 없는 경우가 있음
        - 1에서 부터 후위 순회로 풀어야 했음
- 답 보고 푼 풀이
```python3
import sys

sys.setrecursionlimit(10**8)
input = sys.stdin.readline
n = int(input())
tree = [[] for _ in range(n+1)]
node = [[], [0,0]]

for i in range(2, n+1):
    [t,a,p] = input().split()
    tree[int(p)].append(i)
    node.append([t,int(a)])

def dfs(cur):
    amount = 0
    for i in tree[cur]:
        amount += dfs(i)
        
    if node[cur][0] == 'W':
        amount -= node[cur][1]
        amount = max(amount, 0)
    else:
        amount += node[cur][1]
        
    return amount

print(dfs(1))
```
