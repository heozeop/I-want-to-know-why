---
title: "Algorithm Boj 2568"
date: 2023-10-30T22:27:52+09:00
draft: false

categories:
- algorithm
---

### 문제
- [안전 영역](https://www.acmicpc.net/problem/2468)

## 시도
### 생각 나는 방법
- 핵심
    1. 물뿌리기
    1. 안전 영역 구하기
    1. 1-2 가장 큰거까지 반복
- 물뿌리기
    - n,n 찾아 보기
    - N^2
- 안전 영역 구하기
    - dfs로 돌면서 찾기
    - N^2
- 가장 큰거 까지 반복
    - M
- 시간 복잡도
    - N^3
- 필요 상수
    - diresctions
- 필요 변수
    - visit map
    - input
- 코드
    ```python
    import sys
    sys.setrecursionlimit(100000)

    directions = [
        [0, 1],
        [0, -1],
        [1, 0],
        [-1, 0]
    ]

    AVAILABLE = -1
    NOT_AVAILABLE = -2

    def init(visit_map, source_map, n):
        for i in range(n):
            for j in range(n):
                visit_map[i][j] = source_map[i][j]


    def watering(visit_map, target_value, n):
        global AVAILABLE,NOT_AVAILABLE
        for i in range(n):
            for j in range(n):
                if visit_map[i][j] <= target_value:
                    visit_map[i][j] = NOT_AVAILABLE
                else:
                    visit_map[i][j] = AVAILABLE

    def dfs(x,y, visit_map, target_value, n):
        global AVAILABLE
        
        for i in range(4):
            nx = x + directions[i][0]
            ny = y + directions[i][1]
            
            if nx < 0 or ny < 0 or nx >= n or ny >= n:
                continue
            if visit_map[nx][ny] != AVAILABLE:
                continue
            visit_map[nx][ny] = target_value
            dfs(nx,ny, visit_map, target_value, n)

    def find_safe_place(visit_map, n):
        count = 1
        for i in range(n):
            for j in range(n):
                if visit_map[i][j] == AVAILABLE:
                    visit_map[i][j] = count
                    dfs(i,j, visit_map, count, n)
                    count += 1
        return count - 1
                    

    if __name__ == "__main__":
        n = int(input())
        source_map = []
        for _ in range(n):
            source_map.append(list(map(int, input().split())))
        
        max_value = 0
        for i in range(0, 100):
            visit_map = [[0] * n for _ in range(n) ]
            init(visit_map, source_map, n)
            watering(visit_map, i, n)
            max_value = max(max_value, find_safe_place(visit_map, n))
            
        print(max_value)
        
    ```
- 놓친 점
    1. 물 높이 범위
        - 범위 써두기

