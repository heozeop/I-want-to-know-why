---
title: "Algorithm Boj 2667"
date: 2023-10-23T20:59:09+09:00
draft: false

categories:
- algorithm
---

### 문제
- [단지 번호 붙이기](https://www.acmicpc.net/problem/2667)

## 주요 로직을 생각해보자.
### 1번: 막생각 나는 대로 | O(n^3)
- 2차원 배열을 입력 받는다.
- 방문 여부를 const로 설정한다.
- 2차원 배열을 순회한다.
    - 1이 아니면 통과한다.
    - 상하좌우 탐색한다.
    - dfs로 집 수를 찾는다.
    - 집수 list에 저장한다.
- 집수 list를 정렬한다.
- 집수를 출력한다.

### 구현
```python

TARGET = '1'
VISITED = -1
cost_list = []
directions = [
    [0, 1],
    [0, -1],
    [1,0],
    [-1,0]
]

def dfs(x,y):
    global n
    
    map_list[x][y] = VISITED
    
    localCount = 1
    for i in range(4):
        nx = x + directions[i][0]
        ny = y + directions[i][1]
        
        if nx < 0 or ny < 0 or nx >= n or ny >= n:
            continue
            
        if map_list[nx][ny] != TARGET:
            continue
        
        localCount += dfs(nx,ny)
        
    return localCount

for i in range(n):
    for j in range(n):
        if map_list[i][j] != TARGET:
            continue
        cost_list.append(dfs(i,j))

cost_list.sort()
print(len(cost_list))
for i in cost_list:
    print(i)
```

---
### 틀린 점
- sorted()는 pure function이다.
