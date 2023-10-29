---
title: "Algorithm Boj 2638"
date: 2023-10-29T18:30:34+09:00
draft: false

categories:
- algorithm
---

### 문제
- [치즈](https://www.acmicpc.net/problem/2638)

## 문제 풀이 하기
### 방법 1 생각
1. 가볍게
    1. 녹을 치즈 찾기
    1. 녹이기
    1. 1,2번 반복
1. 녹을 치즈 찾기
    1. bfs로 치즈 아닌 부분 찾기
    1. 윤곽중 2면 이상 치즈 아닌 부분 맡닿은거 마크
    1. 지우기
1. 필요한 상수
    1. direction
1. 필요한 변수
    1. 치즈 아닌 부분 저장할 list
    1. 치즈 아닌 부분 탐색할 queue
    1. 마크 저장할 list
    1. 맵 저장할 list
1. 시간 복잡도
    1. bfs N^2
    1. N일듯
    1. => N^3
--- 
1. 코드
    ```python3
    from collections import deque

    n, m = list(map(int, input().split()))
    source_map = []
    for _ in range(n):
        source_map.append(list(map(int, input().split())))
        
    directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]

    target = 0
    target_value = -1

    def find_non_cheese_section(visit_map):
        global n, m, target, target_value    
        
        queue = deque()
        queue.append([0,0])
        while len(queue) > 0:
            x, y = queue.popleft()
            
            for i in range(4):
                nx = x + directions[i][0]
                ny = y + directions[i][1]
                
                if nx < 0 or ny < 0 or nx >= n or ny >= m:
                    continue
                if source_map[nx][ny] != target:
                    continue
                if visit_map[nx][ny] == target_value:
                    continue
                
                visit_map[nx][ny] = target_value
                queue.append([nx,ny])
                
    def find_melt_cheese_list(visit_map):
        global n, m, target, target_value
        melt_pos_list = []
        
        for i in range(n):
            for j in range(m):
                if visit_map[i][j] == target_value:
                    continue
                count = 0
                for k in range(4):
                    x = i + directions[k][0]
                    y = j + directions[k][1]
                    
                    if x < 0 or y < 0 or x >= n or y >= m:
                        continue
                    if visit_map[x][y] == target_value:
                        count += 1
                if count >= 2:
                    melt_pos_list.append([i,j])

        return melt_pos_list

    def melt_cheese(melt_pos_list):
        for x,y in melt_pos_list:
            source_map[x][y] = 0

    def is_no_cheese_left():
        for i in range(n):
            for j in range(m):
                if source_map[i][j] == 1:
                    return False
        return True

    count = 0
    while not is_no_cheese_left():
        visit_map = [[0] * m for _ in range(n)]
        find_non_cheese_section(visit_map)
        melt_pos_list = find_melt_cheese_list(visit_map)

        melt_cheese(melt_pos_list)
        count += 1

    print(count)
    ```
1. 발생 오류
    1. index 오류
        - 방향 탐색시 상하좌우 제한 조건 넣기
        - N,M 주의하기
