---
title: "Algorithm Boj 15686"
date: 2023-10-13T19:45:06+09:00
draft: true

categories:
- algorithm
---

## 핵심 아이디어
1. NM짜리 이차원 배열을 이용해 최소 치킨 거리를 구한다.
1. 백트랙을 이용해 M개 집 치킨거리를 구한다.

## 구현
```python
n, m = list(map(int, input().split()))

city = []
for _ in range(n):
    city.append(list(map(int, input().split())))

houseList = []
chickenList = []

for i in range(n):
    for j in range(n):
        if city[i][j] == 1:
            houseList.append([i,j])
        if city[i][j] == 2:
            chickenList.append([i,j])

houseChicken = [[0] * len(chickenList) for _ in range(len(houseList))]
for i in range(len(houseList)):
    for j in range(len(chickenList)):
        houseChicken[i][j] = abs(houseList[i][0] - chickenList[j][0]) + abs(houseList[i][1] - chickenList[j][1])

chickenDistance = 123456789
def bfs(index, choosen):
    global chickenDistance
    if index > len(chickenList):
        return
    
    if len(choosen) == m:
        minVal = 0
        for i in range(len(houseList)):
            localMin = 123456789
            for j in choosen:
                localMin = min(houseChicken[i][j], localMin)
            minVal += localMin
        chickenDistance = min(minVal, chickenDistance)
        return
    
    
    bfs(index + 1, choosen)
    choosen.append(index)
    bfs(index + 1, choosen)
    choosen.pop()
    
bfs(0, [])
print(chickenDistance)
```

### 아쉬운 점
- 범위 실수가 많았다.
    - iteration 도는 횟수등 주의하자.

### 잘한점
- 시간 복잡도를 먼저 구했다.


