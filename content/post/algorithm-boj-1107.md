---
title: "[알고리즘] Algorithm Boj 1107"
date: 2023-05-26T05:12:05+09:00
draft: false

categories:
- algorithm
tags:
- 1107
- bruteforce
keywords:
- 1107
- bruteforce
---

> TL;DR
1. GOLD4 문제 왜틀리지 시전
1. 틀린 케이스를 찾아버리고야 말테다
1. 그렇구나. 정확한 사고로 풀어야 겠다

# 오늘의 문제
![boj-1107](https://github.com/heozeop/I-want-to-know-why/assets/29042329/0c4ab650-c601-4802-916f-ecdbe1818f3b)

## 뭔일인가.

### 논리적인 접근
위 문제는 특정한 값 위나 아래로 접근하며 가장 빠르게 도달하는 경우의 수를 구하는 문제입니다.
이러한 특성에 근거해, 목표로 하는 값의 2배로 접근시 충분히 원하는 결과를 얻을 것이라고 생각했습니다.

*2번째 제출 까지는 말이죠*

- 틀린답
<details>
    <summary>틀린 답</summary>
    <p>
    ```C++
    #include <climits>
    #include <iostream>
    #include <algorithm>
    #include <cstdlib>
    #include <queue>
    #include <vector>
    #include <limits.h>
    #include <set>
    #include <cstring>

    using namespace std;

    vector<int> allowed;
    int targetChannel;
    int visited[1000001];
    int m;
    int count(int); // 특정 지점으로 가는데까지 버튼 누르는 수 계산
    void findCombinations(int, vector<int>&); // 1~제한까지 숫자 조합 생성

    void input();
    void solve();

    int main(void) {
      input();
      solve();
      return 0;
    }

    void input() {
      cin >> targetChannel >> m;

      int temp;
      set<int> blocked;
      for(int i = 0; i < m; ++i) {
        cin >> temp;
        blocked.insert(temp);
      }

      for(int i = 0; i < 10; ++i) {
        if(blocked.find(i) != blocked.end()) {
          continue;
        }

        allowed.push_back(i); 
      }

      return;
    }

    void solve() {
      if(m == 10) {
        cout << abs(targetChannel - 100);
        return;
      }

      vector<int> arr;

      findCombinations(0, arr);

      sort(arr.begin(), arr.end());

      int localLow = INT_MAX;
      for(int i = 0; i < arr.size(); ++i) {
        localLow = min(count(arr[i]), localLow);
      }

      cout << min(localLow, abs(targetChannel - 100));

      return;
    }

    void findCombinations(int n, vector<int>& arr) {
      if(n * 10 >= targetChannel * 10) { // 여기가 틀림
        return;
      }

      if (visited[n]) {
        return;
      }

      visited[n] = 1;
      int nn;
      for(int i = 0; i < allowed.size(); ++i) {
        nn = n * 10 + allowed[i];

        if(nn <= targetChannel * 10) { // 여기가 틀림
          arr.push_back(nn);
          findLowerBound(nn, arr);
        }
      }

      return;
    }

    int count(int on) {
      return abs(targetChannel - on) + to_string(on).length();
    }

</p>
</details>

### 답을 찾아보았다.
하도 안풀려서 답을 보니 그냥 100만으로 제한을 풀고 돌리더군요.
애석하게도... 답이 되었답니다.

저는 이해할 수 없었습니다. 그래서 테스트 케이스를 찾기 시작했습니다.
열심히 돌려 888888에서는 되고, 888887에서는 실패를 했습니다.
아하! 모먼트 였습니다.

- page가 2개지요!
    ![boj 문제 제출 기록](https://github.com/heozeop/I-want-to-know-why/assets/29042329/ae7e2b65-9d13-430d-b7cf-da03d5bba3a5)

### 고려할 경우의 수
이 문제에서 고려해야 하는 문제는 3가지 입니다.
이중 큰값에서 내려올때 저 멀리서 뛰어오는 값이 있을 것입니다. 이 값은 100만 보다는 작을 것입니다.
즉 어떠한 가정을 세우기 보다 100만이라는 숫자에 집중할 필요가 있었다는 것입니다.
1. 최초 시작과의 거리 
1. 큰값에서 내려올때 거리
1. 작은 값에서 내려올때 거리

## 앞으로는
1. 추정이 아닌 정확한 근거에 기반해서 문제에 접근합니다.
1. 논리 구조가 빈약하면 선택을 보류합니다.

