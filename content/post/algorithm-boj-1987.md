---
title: "Algorithm Boj 1987"
date: 2023-11-07T21:23:34+09:00
draft: false

categories:
- algorithm
---

### 문제
1. [알파벳](https://www.acmicpc.net/problem/1987)

## 해결 방법을 생각해 보자.
### 방법 1. 생각나는 대로
- 핵심
    - 백트랙킹으로 visit을 탐색하면서 돈다.
- 자세하게
    1. dfs로 이동 가능한 영역 접근한다.
    1. 여러번 하지 않게 접근한 횟수를 기록한다.
    1. 갈데 없으면 비교하고 하나 뺀다.
- 필요 상수
    1. directions
- 필요 변수
    1. visit
    1. visitBoard
    1. n,m
    1. board
- 시간 복잡도
    1. O(4 * 26)
        - 4방향, 최대 26개

---
- 실패한 원인
    1. in으로 확인하는데 시간이 오래 걸린다.
    1. dp처럼 풀려고 했다.
- 코드
```python
n,m = list(map(int, input().split()))

board = []
for _ in range(n):
    board.append(list(input()))
directions = [
    [0,1],[0,-1],[1,0],[-1,0]
]

check = [0]* 26

max_value = 0
check[ord(board[0][0]) - 65] = 1


def backtrack(x,y,count):
    global n,m,board,max_value
    
    for i in range(4):
        nx = x + directions[i][0]
        ny = y + directions[i][1]
        
        if nx < 0 or ny < 0 or nx >= n or ny >= m:
            continue
        
        if check[ord(board[nx][ny]) - 65] == 1:
            continue
        
        check[ord(board[nx][ny]) - 65] = 1
        backtrack(nx,ny,count + 1)
        check[ord(board[nx][ny]) - 65] = 0
        
    max_value = max(max_value, count)
    
backtrack(0,0,1)
print(max_value)
        
```
