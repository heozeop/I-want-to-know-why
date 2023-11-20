---
title: "Algorithm Boj 14502"
date: 2023-10-23T22:07:43+09:00
draft: false

categories:
- algorithm
---

### 문제
- [연구소](https://www.acmicpc.net/problem/14502)

## 접근 방법을 생각해보자.
### 방법 1: 생각나는 것: ~47분
- 바이러스 위치 확인하기
- 빈칸 위치 확인하기
- 빈칸 nC3 으로 위치뽑기
    - 바이러스 위치 기준으로 돌면서 dfs로 맵 채우기
    - 전체 - 바이러스 - 벽수로 빈값 구하기
    - 빈값 크기 비교 하기
- (n * m)C3 * (n * m) => 약 266만

- 구현
    ```python
    directions = [
        [0,1],
        [0,-1],
        [1,0],
        [-1, 0]
    ]

    def dfs(x,y, visited):
        global n,m, lab_map;
        
        for i in range(4):
            nx = x + directions[i][0]
            ny = y + directions[i][1]
            
            if nx < 0 or ny < 0 or nx >= n or ny >= m:
                continue

            if lab_map[nx][ny] != 0:
                continue
                
            if visited[nx][ny] != 0:
                continue
            
            visited[nx][ny] = 1
            dfs(nx,ny,visited)

    virus_location_list = []
    empty_location_list = []
    wall_count = 0

    for i in range(n):
        for j in range(m):
            if lab_map[i][j] == 2:
                virus_location_list.append([i,j])
            elif lab_map[i][j] == 1:
                wall_count += 1
            elif lab_map[i][j] == 0:
                empty_location_list.append([i,j])
                
    max_count = 0
    len_empty_location_list = len(empty_location_list)
    for i in range(len_empty_location_list):
        ix,iy = empty_location_list[i]
        lab_map[ix][iy] = 1 
        for j in range(i + 1, len_empty_location_list):
            jx,jy = empty_location_list[j]
            lab_map[jx][jy] = 1
            for k in range(j + 1, len_empty_location_list):
                kx,ky = empty_location_list[k]
                lab_map[kx][ky] = 1 
                visited = [[0] * m for _ in range(n)]
                for [x,y] in virus_location_list:
                    visited[x][y] = 1
                    dfs(x,y,visited)
                    
                count_zero = 0
                for x in range(n):
                    for y in range(m):
                        if visited[x][y] == 0:
                            count_zero += 1
                max_count = max(max_count, count_zero - 3 - wall_count)
                lab_map[kx][ky] = 0        
            lab_map[jx][jy] = 0
        lab_map[ix][iy] = 0
    print(max_count)
    ```
- 더 깔끔한 코드
    - [14502번 : 연구소 (python)](https://jie0025.tistory.com/209)
---
### 알게된 것
- 200만대면 2초안에 풀린다.

