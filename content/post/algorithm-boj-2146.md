---
title: "Algorithm Boj 2146"
date: 2023-10-26T20:23:08+09:00
draft: false

categories:
- algorithm
---

### 문제
- [다리 만들기](https://www.acmicpc.net/problem/2146)

## 문제를 생각해 보자.
### 1. 생각나는 대로: 
- 대강
    - 대륙은 갈 수 없는 곳
    - 대륙 옆이 길이 1로 설정
    - bfs로 모두 돌면서 다른 대륙 도달하기
- 자세히
    - 대륙 별로 대륙 가를 1자로 한다.
        - visit을 나타낼때 출발지, 거리를 나타낸다.
    - 만난 곳이라면 거리를 더해서 min에 저장한다.
    - min을 출력한다.
- 시간 복잡도
    - N^2
        - 0인 곳 + 상하좌우에 대륙있는 곳
            - n^2
        - 탐색
            1. 같은 대륙 출발
                - 대륙
                    - 통과
                - 바다
                    - 짧은 데
            1. 다른 대륙
                - 계산
- 시도 1 문제
    1. 대륙이 통과하면 만나지를 못한다.
    1. 초기화를 잘못한다.
- 시도 문제
    1. 구체적인 구현 사례에서 오류가 꽤 있었다.
    1. 채워 나가는 방식으로 해보기
    