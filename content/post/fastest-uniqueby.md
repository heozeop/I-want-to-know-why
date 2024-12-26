---
title: "가장 빠른 unique by를 만들어 보자"
date: 2024-12-25T23:10:42+09:00
draft: false

categories:
- nodejs
tags:
- nodejs
keywords:
- fastest unique by
---

# TL;DR
1. native set, lodash, ImmutableJS의 Set을 이용한 속도/메모리 사용량/CPU 사용량을 봅니다.
1. CPU 사용량이 많아지는 것에 대한 걱정이 없다면 ImmutableJS의 Set을 활용한 uniqueBy를 구현해 사용하세요

# 꽤 많은 데이터에 대해서 uniqueBy를 빠르게 돌리고 싶다.
## 왜 와이
### 백만개 정도 되는 데이터를 돌릴 필요가 발생했다.
백만개 row 정도 되는 json array에 대한 uniqueBy를 할 필요가 생겼습니다.
실제로 백만개 row에 대해서 테스트는 하기 어려워서 그냥 native set을 이용해 개발했습니다.
개발을 하고 보니, 더 빠른 구조가 있을지 궁금해 졌습니다.
chatGPT에 물어보니, TypedArray를 사용해 직접 구현하는 방법과 ImmutableJS를 이용해 구현하는 방법을 제안해 주었습니다.

### 직접 Set을 구현하는 것은 포기했습니다
직접 구현하는 것을 고려할 때 줄이고자 한 비용은 자료구조의 resizing 비용이었습니다. resizing이 되지 않기 위해서는 미리 데이터 크기를 맞춰서 set을 구현해야 합니다. native set에서는 bucket과 load factor를 지정할 수 없기 때문에 TypedArray를 내부적으로 가지는 set의 구현을 고려했습니다.

첫번째 문제는 key를 hash 하는 방법이었습니다. resizing하는 시간조차 아끼려는 시도이기 때문에 hash 함수는 빠르고 충돌이 적은 것을 우선으로 검토하였습니다. chatGPT를 활용해 murmur3, cityHash64를 찾았고, 해당 hash를 이용해 저장할 수 있도록 초안을 구성했습니다.

두번째 문제는 데이터를 적재하는 것이었습니다. TypedArray를 이용하더라도 결국 데이터를 넣을때는 index를 활용해야 합니다. 이때 활용할 수 있는 방법은 Set 내부에 hashtable을 생성해 참조하는 방법, 직접 index를 구해 넣는 방법이 있습니다. 내부에 hashtable을 생성해 참조하는 방법을 활용하면 resizing 비용을 줄이기 위해 set을 직접 만든다는 의미가 퇴색된다고 생각했습니다. 직접 index를 hash로부터 구하는 방법은 충돌을 회피할 수 없다는 문제가 발생한다는 문제가 있었습니다.

저는 이쯤에서 set을 직접 구현하는 것을 포기하였습니다. 떠오른 2가지 방법으로 native 보다 빠르게 만들 수는 없겠다는 생각이 들었기 때문입니다. 추후 Set들의 내부 구현들을 찾아보면서 경험치를 더 쌓고 난 후 재실험을 하자고 결정했습니다.

# 실험
## 목적
### 빠른 uniqueBy 구현체를 찾자
실험군으로 native set, lodash, ImmutableJS의 Set를 선정했습니다.

모니터링 요소로는 시간, 메모리 사용량, cpu 사용량으로 결정했습니다.

### 고려하지 않은 것
- node 내부 동작에 의한 시간 차
시간을 측정하는 함수를 process의 hrtime을 사용해 직접 구현했는데 node 내부 동작에 의해 발생할 수 있는 일부 시간차는 무시 했습니다.
- 모니터링이 실시간이 아닌 것
grafana 세팅에 힘을 쏟고 싶지 않아서 5초에 한번 수집되는 기본 값을 사용했습니다. 

## 환경
### Docker를 이용해 memory와 cpu를 제한
- memory 2GB, cpu 0.25
최초에는 memory 512MiB만 사용하고자 했습니다. 하지만 실험을 진행하면서 만난 이슈를 대응하기 위해 momory 2GB로 확장해 넣었습니다.

### 데이터
데이터 구조는 아래와 같이 해서 JSON 파일로 만들어 처리했습니다.
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

### 모니터링
- 메모리와 cpu 사용량
    prometheus, grafana를 app과 함께 docker compose로 띄워 기본적인 사용량을 추적하였습니다.
- 시간
    직접 함수의 실행 시간을 측정하는 함수를 작성했습니다.

### 실행 배경
youtube를 비롯한 여러 탭이 돌고 있는 Macbook Air M3 입니다.

## 방법
### manual한 실행
nestjs를 이용해 엔드포인트를 직접 호출하는 것으로 실행하였습니다.

### 실험군 코드
- native Set을 이용한 unique by
    ```typescript
    function uniqueBy(array, key) {
      const seen = new Set();
      return array.filter(item => {
        const key = item[key];
        if (seen.has(key)) {
          return false;
        }
        seen.add(key);
        return true;
      });
    }
    ```

- lodash의 uniqBy를 이용한 unique by
    ```typescript
    import { uniqBy } from 'lodash';

    function uniqueBy(array, key) {
      return uniqBy(array, key);
    }
    ```
- ImmutableJs Set을 이용한 unique by
    ```typescript
    function uniqueBy(array, key) {
      let seen = Set();
      return array.filter(item => {
        const uniqueKey = item[key];
        if (seen.has(uniqueKey)) {
          return false;
        }
        seen = seen.add(uniqueKey);
        return true;
      });
    }
    ```
## 진행
### 기본 메모리 사용량
![기본 메모리 사용량](https://github.com/user-attachments/assets/fd7ff9d0-ea9b-411d-919d-fad4d4768f22)

### 기본 CPU 사용량
![스크린샷 2024-12-25 오후 8 40 52](https://github.com/user-attachments/assets/d42e01f7-02f8-4122-b6ea-5477c55789be)
삐죽 튀어나온 부분은 json 데이터를 읽는 과정에서 string을 json으로 parsing하는데 사용된 것으로 추정

## 진행중 만난 이슈
### 데이터 생성
1. 한번에 저장하는 건 불가했다.
    한번에 100만개 데이터를 생성하려고 최초에는 writeFileSync를 이용해 저장하려고 했습니다. 하지만 `JSON.stringify` 함수에서 `RangeError`가 발생([ref][1])해 writable stream을 열어서 데이터를 저장하는 방법으로 처리하였습니다.

### 모니터링 과정
1. grafana 대시보드의 잘못된 선정
    최초에는 이름만 보고 node-exporter를 사용하려 했습니다. 하지만 제 목적은 nodejs process의 사용량만을 측정하는 것이기 때문에 nodejs application report를 사용해야 했죠. 대시보드가 잘못되었다는 것을 판단하는데 10분 정도 걸렸던 것 같습니다.

### 코드
1. data를 json 파일로 바로 읽을 수 없음
    typescript 옵션 중 `resolveJsonModule`을 이용해 `data.json` 파일을 직접 참조하려 했습니다. 하지만 import할 때도 size 제한이 있었고([ref][1]) 이를 해결하고자 readable stream을 열어서 파일을 참조하였습니다.
1. data를 읽는데 시간이 오래 걸림
    데이터 파일이 거의 700MB가 되다보니 요청마다 읽는 것은 무리라고 생각하게 되었습니다. 이에 대한 해결책으로 nestjs의 `OnModuleInit` 인터페이스를 controller 에서 구현해 미리 로딩해서 가지고 있게 만들었습니다.
1. http timeout이 발생함.
    실험을 진행하는 과정에서 nestjs의 설정인지, docker의 설정인지, insomnia의 설정인지 timeout 오류가 발생했습니다. `server.settimeout(0)`으로는 해결이 안되길래 일단 큰 문제가 아니라고 생각해 넘겼습니다.

## 결과
### 응답 시간
|  | 1 | 2 | 3 | 4 | 5 | 평균 |
| --- | --- | --- | --- | --- | --- | --- |
| set | 7.089213796 | 6.783025795 | 24.56661764 | 8.300059212 | 9.093887211 | 11.16656073 |
| lodash | 14.29638401 | 5.456563836 | 20.20556684 | 5.204739335 | 27.48573922 | 14.52979865 |
| immutableSet | 2.730608459 | 2.10582821 | 1.275754 | 1.556768792 | 1.627140375 | 1.859219967 |

### 메모리 사용량
- 정확하게 측정하지 못함.

### CPU 사용량
- Immutable Set은 약 20 ~ 25% 정도
- lodash는 약 8 ~ 13% 정도
- native set은 약 1 ~ 5% 정도

## 결과 해석
### 속도 관점에서 가장 좋은 uniqueBy는 
Immutable JS의 Set을 이용한 uniqueBy

![chatGPT/native set와 Immutable set의 차이](https://github.com/user-attachments/assets/b8c4085d-4961-451e-ba8f-88bd29167efd)

### CPU 사용량 관점에서 가장 좋은 uniqueBy는
Native set을 이용한 uniqueBy

### memory 사용량
비등비등하나 MB단위로 소모한다는 점은 주목할만 합니다.

## 아쉬웠던 점
### 모니터링 설정이 실시간이 아닌 것
좀 더 빠르게 했으면 메모리나 CPU 사용량을 보다 정밀하게 측정했을 것 같은 아쉬움이 남습니다.

### 직접 구현을 하지 않은 것
나중에 시간을 써서 확인해 보면 좋을 것 같다는 생각이 듭니다.

# 다음에 할 것
## resize 비용에 대한 차이 확인
chatGPT에 물어보니 초기에 Map에 데이터를 먼저 넣어두고 하면 resize에 대한 비용이 줄어든다고 하더 군요. 이를 직접 호출하는 것과 비교해 테스트 해보면 좋을 것 같아 다음 실험으로 고려하고 있습니다.


[1]: https://github.com/nodejs/node/issues/35973