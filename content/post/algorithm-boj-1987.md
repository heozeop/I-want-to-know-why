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
    1. ON^2

