---
title: "Algorithm Boj 16236"
date: 2023-10-24T21:30:46+09:00
draft: false

categories:
- algorithm
---

### 문제
- [아기 상어](https://www.acmicpc.net/problem/16236)

## 해결 방법을 생각해 보자.
### 방법 1: 생각 나는 대로:
- 기본 로직
    - 물고기 크기/위치를 기준으로 정렬한다.
    - 먹을 수 있는 물고기만 담는다.
    - 거리를 계산한다.
    - 가장 짧은 거리의 물고기만 먹는다.
    - 먹은 수대로 상어 크기를 올린다.
- 핵심 로직
    - 상어의 성장 여부 확인
    - 먹을 수 있는 물고기만 담기
    - 거리 계산하기
        - bfs로 탐색하기
- 글로벌 변수
    1. 먹은 물고기 수
        - 상어 성장 여부 판단
    1. 상어 크기
    1. 상어 위치
    1. 시간
- 순차 로직
    1. 상어 위치 찾기
    1. bfs탐색하며 왼 위 아래 오른쪽 순 돌기
    1. 가장 먼저 탐색 된 상어 크기보다 작은 물고기 섭취
        - 물고기 섭취 불가시 종료
    1. 상어 성장 여부 판단
    1. 이동 시간 기록
    1. 1 - 4 반복

- 코드
    ```python
    INF = 1e9
    ate_count = 0
    shark_size = 2
    shark_pos = []
    happy_time = 0
    directions = [
        [-1, 0],
        [0, -1],
        [1,0],
        [0, 1],
    ]

    for i in range(n):
        for j in range(n):
            if n_space[i][j] == 9:
                shark_pos = [i,j]
                n_space[i][j] = 0


    def bfs(x,y):
        global n, shark_size,n_space
        queue = deque()
        queue.append([x,y])
        visited = [[-1] * n for _ in range(n)]
        visited[x][y] = 0
        
        while len(queue) > 0:
            cur_x, cur_y = queue.popleft()
            
            for i in range(4):
                nx = cur_x + directions[i][0]
                ny = cur_y + directions[i][1]
                
                if nx < 0 or ny < 0 or nx >= n or ny >= n:
                    continue
                if n_space[nx][ny] > shark_size:
                    continue
                if visited[nx][ny] != -1:
                    continue
                
                visited[nx][ny] = visited[cur_x][cur_y] + 1
                queue.append([nx,ny])
        return visited

    def checkMin(visited):
        global shark_size
        
        x,y = 0, 0
        min_distance = INF
        for i in range(n):
            for j in range(n):
                if visited[i][j] == -1:
                    continue
                if n_space[i][j] == 0 or n_space[i][j] >= shark_size:
                    continue
                if visited[i][j] >= min_distance:
                    continue
                min_distance = visited[i][j]
                x, y = i, j
                
        if min_distance == INF:
            return False
        
        return x, y, min_distance
                    

    while(True):
        result = checkMin(bfs(shark_pos[0], shark_pos[1]))
        if result == False:
            break;

        nx,ny,time = result

        shark_pos = [nx, ny]
        happy_time += time
        
        ate_count += 1
        n_space[nx][ny] = 0
        
        if ate_count == shark_size:
            shark_size += 1
            ate_count = 0

    print(happy_time)
    ```

---
### 놓친 점
1. 시간 복잡도를 계산하지 않았다.
1. 가장 위/왼쪽이 순회를 통해 탐색이 가능할 것이라 생각했다.
    1. 가다 막히는 경우가 있기 때문에 항상 마름모꼴로 시간이 상승하지 않음을 간과했다.

