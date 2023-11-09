---
title: "Algorithm Boj 11660"
date: 2023-11-09T21:05:23+09:00
draft: false

categories:
- algorithm
---

### 문제
- [구간 합 구하기5](https://www.acmicpc.net/problem/11660)

### 문제 풀이
- 핵심
    - 구간합 사각으로 덧셈하기
    - V_i_j = V_i-1_j + V_i_j-1 - V_i-1_j-1
    - Sum_x2_y2 - sum_x1_y2 - sum_x2_y1 + sum_x2_y2

- 자세히
    - 변수
        - n, m, table_map, sum_map
- 코드
```python3
import sys
input = sys.stdin.readline

n,m = list(map(int, input().split()))

table_map = []
for _ in range(n):
    table_map.append(list(map(int,input().split())))
    
sum_map = [[0] * n for _ in range(n)]

for i in range(n):
    for j in range(n):
        sum_map[i][j] = table_map[i][j]
        
        if i - 1 >= 0:
            sum_map[i][j] += sum_map[i-1][j]
        if j - 1 >= 0:
            sum_map[i][j] += sum_map[i][j-1]
        if i - 1 >= 0 and j - 1 >= 0:
            sum_map[i][j] -= sum_map[i-1][j-1]
            
def sum_val(x1,y1,x2,y2):
    sum_value = sum_map[x2][y2]
    if x1 - 1 >= 0:
        sum_value -= sum_map[x1 - 1][y2]
    if y1 - 1 >= 0:
        sum_value -= sum_map[x2][y1 - 1]
    if x1 - 1 >= 0 and y1 - 1 >= 0:
        sum_value += sum_map[x1 - 1][y1-1]
    return sum_value

for _ in range(m):
    x1,y1,x2,y2 = list(map(int, input().split()))
    
    print(sum_val(x1 - 1,y1 - 1,x2 - 1,y2 - 1))
    
```

- 주의할 점
    1. bigint
    1. 먼저 생각 다 끝내고 하기

