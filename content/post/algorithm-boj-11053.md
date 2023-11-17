---
title: "Algorithm Boj 11053"
date: 2023-11-17T22:04:28+09:00
draft: false

categories:
- algorithm
---

### 문제
- [가장 긴 증가하는 부분 수열](https://www.acmicpc.net/problem/11053)

### 풀이
- 핵심
    - 정렬된 수열
    - back < new => append
    - back > new => bisect_left(arr, new + 1)
- 구체적인 로직
    - bisect_left
    - n개 int로 받기
    - arr 받기
    - arr sort
    - longest_arr = [0]
    - for i in arr:
        - back < i => append
        - back > i => arr\[bisect_left(arr, i + 1)\] = i
    - print(len(longest_arr) - 1)
- 실패
    - 기억에 따라 증가하기만 해야한다고 생각해 같은값을 대체했다.
- 코드
```python
from bisect import bisect_left

n = int(input())
arr= list(map(int, input().split()))

longest_arr = [arr[0]]

for i in arr:
    if longest_arr[-1] < i:
        longest_arr.append(i)
    elif longest_arr[-1] > i:
        longest_arr[bisect_left(longest_arr, i)] = i
        
print(len(longest_arr))
```

