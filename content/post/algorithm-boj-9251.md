---
title: "Algorithm Boj 9251"
date: 2023-11-21T20:32:14+09:00
draft: true

categories:
- algorithm
---

### 문제
- [로봇 조종하기](https://www.acmicpc.net/problem/2169)

### 풀이
- 핵심
    - dfs를 돈다.
    - 경로를 dp한다.
- 구체적인 풀이
    - n, m을 받는다.
    - arr를 받는다.
    - dp를 -101로 초기화 한다. (lower bound)
    - 상좌우 array를 만든다.
    - dfs를 만든다.
        - dp를 두고 dp에 값이 있으면 리턴한다.
        - 없으면 상좌우에 해당하는 만큼 iteration 돈다.
        - dp에 각 방향 따라 max값을 저장한다.
        - dp 값이 없으면 현재 위치값만 넘긴다.
        - dp 값 + 현재 위치값 넘긴다.
    - dp[0][0]을 출력한다.
- 실패
    - 방향에 따라서 dp를 별도로 주어야 한다.
        - 먼저 갔다고 장땡이 아니라 반대 방향으로도 갈 수 있다.


