---
title: "Algorithm Boj 14002"
date: 2023-11-19T21:09:20+09:00
draft: true

categories:
- algorithm
---

### 문제
- [가장 긴 증가하는 부분 수열 4](https://www.acmicpc.net/problem/14002)

### 풀이
- 핵심
    - 긴 수열 길이 구하기
        - 순서 바꾸기로 가능
    - 긴 수열 구하기
        - 순서 바뀌어 다시 구해줘야함.
- 실패
    - 순서가 바뀌는 걸 이해하지 못함.
        - 외워서 풀었다는 이야기
- 코드
```python
from bisect import bisect_left

n = int(input())
arr = list(map(int, input().split()))
idx_list = [0] * (n + 1)

longest_array = [arr[0]]

for i, val in enumerate(arr):
    if longest_array[-1] < val:
        longest_array.append(val)
        idx_list[i] = len(longest_array) - 1
    else:
        idx = bisect_left(longest_array, val)
        longest_array[idx] = val
        idx_list[i] = idx

print(len(longest_array))


ans = []
index = len(longest_array) - 1
for i in range(len(arr) - 1,-1,-1):
    if idx_list[i] == index:
        ans.append(arr[i])
        index -= 1

print(*ans[::-1])
```

