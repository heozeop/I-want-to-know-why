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
1. 실험이 적절하지 않아 resize 비용이 속도에 영향을 주는 정도인지는 알 수 없었습니다.
1. Map을 초기에 intialize해서 resize를 1번 일으키는 것보다 그냥 매번 resize를 일으키되 찾는 연산을 작은 데이터 셋에서 하는게 더 좋다는 것만 알게 되었습니다.

# Resizing을 줄이는게 의미 있을까?
## 뭔말이여
### 100만개의 데이터를 돌리는데 Resizing 비용이 아까웠다.
[지난 실험][1]에서 백만개 item으로 구성된 json array에 대한 uniqueBy를 수행하면서, resizing 비용이 클거라는 생각에 직접 구현을 고민 했었습니다. 하지만 정말로 큰게 맞을까요? chatGPT에게 물어보니 초기에 값을 넣어주는 방식을 채택하면 resize에 들어가는 비용이 줄어든다고 하더군요. 합리적인 말이라고 생각해서 실험해보기로 했습니다.

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
script를 이용해 10번씩 직접 호출하는 방식으로 테스트를 수행했습니다.

실행에 사용한 간단한 스크립트는 아래와 같습니다.
```javascript
const args = process.argv.slice(2);

if (args.length !== 2) {
  console.error('Usage: node data-generator.js <output-file> <data-size>');
  process.exit(1);
}

fetchData(args[0], Number(args[1]));

async function fetchData(path, times) {
  for (let i = 0; i < times; i++) {
    const response = await fetch(path);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    console.log(await response.text());
  }
}

```

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
### 결과
#### 시간

| 유형 | 평균 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 초기 map initialize | 0.5032949335 | 0.4674785|0.822471334|0.379201667|0.356931833|0.829150584|0.398681291|0.366561959|0.629222208|0.398749792|0.384500167 |
| 그냥 map 활용 | 0.382969125용 | 0.508061	| 0.830593126	| 0.262079875	| 0.262612041	| 0.270461417	| 0.28156475	| 0.624479459	| 0.309609208 |	0.244776084	| 0.235454292|

그냥 map을 활용하는게 시간이 0.2초 정도로 평균에서 빠르다는 것을 확인 할 수 있었습니다. 좀 확대 해석 하자면, 초기에 100만개 size를 할당하고 찾는 연산을 하는 것이 매번 resize를 하면서 적은 set에서 찾는 연산을 하는 것보다 느리다는 것을 알 수 있습니다. 즉, initialize를 활용한 resize는 오히려 찾는 연산 시간을 크게 만들어 느리게 된다는 이야기죠!

#### 컴퓨팅 자원
- 초기 map initialize
![초기 map initialize](https://github.com/user-attachments/assets/97947c90-96c6-4df9-8e41-83fdd090ce02)
- 그냥 map만 활용한 것
![그냥 map](https://github.com/user-attachments/assets/a41c76a2-375b-4660-8645-d582e538ca65)

사용한 메모리는 비슷했으나, 초기 map initialize한 경우 cpu 자원을 150%이상 활용했고 그냥 map을 활용한 것은 cpu 자원을 150% 미만 활용했음을 알 수 있었습니다. 이 역시 위 시간과 같이 map을 만들어 찾는 연산 때문에 차이가 발생했다고 보여집니다. 

### 결론
- resize 비용 보다는 큰 데이터가 할당된 상태에서 자원을 찾는 연산이 시간/자원활용에 더 많은 영향을 줍니다.
- typescript/javascript에서는 그냥 map을 직접 활용하는게 더 빠를 수 있습니다.

#### 아쉬운 점
- 실험이 resize만을 테스트하기에는 부적합했습니다.
    - 직접 fixed size map을 구현했다면 더 정확한 비교가 가능했을 것 같아 아쉽습니다.

[1](https://heozeop.github.io/post/fastest-uniqueby/)