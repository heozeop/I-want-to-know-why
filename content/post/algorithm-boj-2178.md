---
title: "Algorithm Boj 2178"
date: 2023-10-23T21:43:03+09:00
draft: false

categories:
- algorithm
---

### 문제
- [미로탐색](https://www.acmicpc.net/problem/2178)

## 해법을 생각해보자.
### 방법 1: 그냥 떠오르는 대로
- bfs로 0,0부터 탐색
- 가장 먼저 N,M 도착하면 출력하기
- n^2
- 구현
    ```python3
    from collections import deque

    directions = [
        [0,1],
        [0,-1],
        [1,0],
        [-1,0],
    ]

    VISITED = -1
    TARGET = '1'

    queue = deque()
    queue.append([0,0,1])
    maze_list[0][0] = VISITED

    while(len(queue) > 0):
        x, y, count = queue.popleft()
        
        if x == n - 1 and y == m - 1:
            print(count)
            break
        
        for i in range(4):
            nx = x + directions[i][0]
            ny = y + directions[i][1]
            
            if nx < 0 or ny < 0 or nx >= n or ny >= m:
                continue
            if maze_list[nx][ny] != TARGET:
                continue
            maze_list[nx][ny] = VISITED
            queue.append([nx, ny, count + 1])
    ```

