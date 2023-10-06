---
title: "Boj 2661"
date: 2023-10-05T18:31:38+09:00
draft: false

categories:
- algorithm
---

# 주요 로직을 생각해 보자.
- 끝에 하나 붙을때 마다 점검해준다.

# 코드
```python3
n = int(input())
s=[]

def isBadSequence(addStr):
    temp = "".join(s) + addStr
    for i in range(1, len(temp) // 2 + 1):
        if temp[-2 * i : -1 * i] == temp[-1 * i :]:
            return True
        
    return False

def dfs():
    global s
    
    if len(s) == n:
        print("".join(s))
        exit()

    for j in range(1,4):
        if isBadSequence(str(j)):
            continue
            
        s.append(str(j))
        dfs()
        s.pop()
    
dfs()
```

# 놓친 점
- 점검 어떻게 할지 구체적으로 생각하지 못함.
- 좀 더 생각하고 풀기 
