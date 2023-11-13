---
title: "Algorithm Boj 2616"
date: 2023-11-13T18:43:44+09:00
draft: true

categories:
- algorithm
---

### 문제
- [소형기관차](https://www.acmicpc.net/problem/2616)

### 해결 방안
1. 핵심
    1. 정렬
    1. 겹치는데만 피해서 2번째 찾기
    1. 1,2번과 안겹치는 3번 찾기
    1. 1번에서 2번 사이 iteration 돌기
    1. 2번에서 3번 사이 iteration 돌기
1. 모듈화
    1. sort
    1. find second
    1. find third
    1. iterate 1 ~ 2
        1. iterate 2 ~ 3

1. 시간 복잡도
    1. O(n^2) + 최적화
---
1. 실패
    1. 주어진 예시가 반례
