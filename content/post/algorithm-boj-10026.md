---
title: "Algorithm Boj 10026"
date: 2023-10-31T08:21:45+09:00
draft: true

categories:
- algorithm
---


### 문제
- [적록 색약](https://www.acmicpc.net/problem/10026)

## 해답 찾기
### 핵심 로직
1. 적록색약이 아닌 경우에 구역 개수 구하기
1. 적록색약인 경우에 구역 개수 구하기
1. 각각 출력하기

### 구역 개수 구하기
1. n ^ 4 > 1억이니 괜찮을 것 같다.
1. 이중 FOR 구문에서 DFS를 돌면서 VISIT_MAP을 활용해 탐색한다.

### 적록색약인 경우
1. G를 R로 바꾸는 전처리를 먼저한다.

### 전체 흐름
1. 맵 복사
    1. 적록 색약인 경우 바꾸기
1. 구역 개수 구하기
1. 출력

### 필요 상수
1. directions

### 필요 변수
1. n : 1 ~ 100
1. source_map : 2차원, 100 * 100
1. visit_map : 2차원, 100 * 100
1. count : 숫자

### 코드
```python
import sys
sys.setrecursionlimit(10000)

DIRECTIONS = [[0,1], [0,-1], [1,0], [-1,0]]
VISITED = 1

def dfs(x,y,target, visit_map, n):
    global VISITED
    
    for i in range(4):
        nx = x + DIRECTIONS[i][0]
        ny = y + DIRECTIONS[i][1]
        
        if nx < 0 or ny < 0 or nx >= n or ny >= n:
            continue
        if visit_map[nx][ny] != target:
            continue
        visit_map[nx][ny] = VISITED
        dfs(nx,ny,target,visit_map,n)

def copy_map(source_map, n,is_red_green_blind):
    visit_map = [[0] * n for _ in range(n)]
    
    for i in range(n):
        for j in range(n):
            visit_map[i][j] = source_map[i][j]
            
            if is_red_green_blind and visit_map[i][j] == 'G':
                visit_map[i][j] = 'R'
    return visit_map

def find_sections(visit_map, n):
    count = 0
    for i in range(n):
        for j in range(n):
            if visit_map[i][j] == VISITED:
                continue
            count += 1
            target =visit_map[i][j]
            visit_map[i][j] = VISITED
            dfs(i,j,target,visit_map,n)
            
    return count
                
if __name__ == "__main__":
    n = int(input())
    source_map = []
    
    for _ in range(n):
        source_map.append(list(input()))
    
    visit_map = copy_map(source_map, n, False)
    section_count = find_sections(visit_map, n)
    print(section_count, end = " ")
    
    visit_map = copy_map(source_map, n, True)
    print(find_sections(visit_map,n))
```
