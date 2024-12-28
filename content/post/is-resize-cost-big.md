---
title: "Map의 Resize 비용은 정말 클까"
date: 2024-12-28T11:35:51+09:00
draft: true

categories:
- nodejs
tags:
- nodejs
keywords:
- fastest unique by
---

# TL;DR
1. 100만개의 item에 대해서 Map을 활용해 자료구조의 resizing 비용이 큰지 비교하는 간단한 실험입니다.

# Resizing을 줄이는게 의미 있을까?
## 뭔말이여
### 100만개의 데이터를 돌리는데 Resizing 비용이 아까웠다.
[지난 실험][1]에서 백만개 item으로 구성된 json array에 대한 uniqueBy를 수행하면서, resizing 비용이 클거라는 생각에 직접 구현을 고민 했었습니다. 하지만 정말로 큰게 맞을까요? chatGPT에게 물어보니 초기에 값을 넣어주는 방식을 채택하면 resize에 들어가는 비용이 줄어든다고 하더군요. 합리적인 말이라고 생각해서 실험해보기로 했습니다. 여기에 더해 resize에 필요한 알고리즘을 V8엔진의 코드를 살펴보며 알아보았습니다.

## Resizing의 과정

## 실험
### 목적
#### resize 비용이 정말 큰지를 실험하자
비교할 대상으로 native Map을 활용한 2개의 uniqueBy 구현체를 사용했습니다. 하나는 최초 Map을 생성할 때 data를 넣어주었고, 하나는 그냥 생성하였습니다. 이때 간과하면 안되는 부분이 전제에서 데이터를 넣어주기 위해 array를 1번 순회한다는 점입니다. 결국 resize를 줄이는 방법을 채택하기 위해서는 필요한 과정이라고 생각하여 별도 분리는 하지 않았습니다.

### 환경
### Docker를 이용해 memory와 cpu 제한
- memory 2GB, cpu 2.0
[지난 실험][1]과 같은 코드베이스를 공유하여 memory는 2GB로 설정했습니다. 다만 cpu는 2.0으로 설정했는데, 지난 실험에서 설정한 0.25로는 초기 실행 속도가 너무 느려 불편했기 때문에 설정했습니다.

### 데이터
데이터 구조는 [이전][1]과 같습니다. 동일한 파일을 사용했습니다.
```typescript
interface Data {
	key: string;
	tempValue1: string;
	tempValue2: string;
	tempValue3: string;
	tempValue4: string;
	tempValue5: string;
	tempValue6: string;
	tempValue7: string;
	tempValue8: string;
	tempValue9: string;
	tempValue10: string;
}
```

#### 모니터링
- 메모리와 cpu 사용량
    prometheus, grafana를 app과 함께 docker compose로 띄워  기본적인 사용량을 추적하였습니다. 지난번의 아쉬움을 만회하기 위해 grafana와 prometheus 설정에 대해 공부해 좀 더 빠르게 (1초에 1번) sync를 맞추도록 설정했습니다.
- 시간
    직접 함수의 실행 시간을 측정하는 함수를 작성했습니다.

### 방법
#### manual한 방법
엔드포인트를 추가해 직접 호출하는 방식으로 실험하였습니다. 다음에는 직접 호출하는 방식 말고 10번 수행해서 평균과 함께 시간을 응답하도록 만들어야 겠습니다. 

#### 실험군 코드
- 초기 Map을 data로 initialize하는 uniqueBy
    ```typescript
     uniqueByMapWithInitialize<T, K extends keyof T>(array: T[], key: K) {
      const seenMap = new Map(array.map((item) => [item[key], false]));
      return array.filter((item) => {
        const keyValue = item[key];
        if (seenMap.get(keyValue)) {
          return false;
        }

        seenMap.set(keyValue, true);
        return true;
      });
    }
    ```
- 그냥 Map을 활용한 uniqueBy
    ```typescript
    uniqueByMap<T, K extends keyof T>(array: T[], key: K) {
      const seenMap = new Map();
      return array.filter((item) => {
        const keyValue = item[key];
        if (seenMap.get(keyValue)) {
          return false;
        }

        seenMap.set(keyValue, true);
        return true;
      });
    }
    ```
### 진행
[1]()