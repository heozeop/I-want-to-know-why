---
title: "노드에서 배치잡 돌리기(feat. child_process, cluster, worker_threads)"
date: 2024-05-16T20:30:55+09:00
draft: false

categories:
- nodejs
tags:
- nodejs
keywords:
- batch job
- concurrent process
---

# TL;DR
1. node에서 배치잡을 돌리고 싶다.
1. child_process, cluster, worker_thread 비교
1. io intensive + cpu intensive일 때의 비교
1. child_process 쓸거
    1. 함수별 사용 시점 정리

# node에서 배치잡을 돌리고 싶다.
## 왜와이
### 쉽고 빠르게 배치작업을 하고 싶다.
일회성 배치작업을 해야 하는 상황이 생겼습니다. 
쉽고 빠르게 익숙한 NodeJS로 배치작업을 하고 싶었습니다.
io와 cpu 모두 사용하는 작업으로, 딱봐도 오래걸리는 작업이었습니다.

### 그냥 해보았습니다.
싱글 스레드로 작업을 진행했습니다.
Promise.all로 일부 데이터만 작업을 진행했더니 20분이 걸렸습니다.
도저히 전체를 한번에 돌릴 수는 없다고 생각이 들었고, 빠르게 처리하는 방법을 찾게 되었습니다.

(퇴근하고 실험하고 있네요 허허)


# child_process, cluster, worker_thread 비교


## child_process
### 기능
자식 프로세스를 생성하는 방법입니다.
spawn, exec, fork 등의 함수를 이용해 프로세스를 생성할 수 있습니다.
(이외 execFile, spawnSync 등의 함수가 존재합니다.[[3]])
기본적인 구분으로는 spawn 과 exec가 있고, fork는 spawn의 특별한 케이스로 본다고 합니다.[[1],[2],[3]]
excec와 spawn은 프로세스를 띄워 command를 실행하는 방식이고, 아래와 같은 차이를 가집니다.


| 기준 | spawn | exec |
|-|-|-|-|
| IPC방식 | stream | buffer |
| return | child process reference | void (promisify하면 {stdout, stderr} 리턴함) |
| 쉘 실행 | x(shell option을 넘기면 실행) | o |

fork 함수는 위에서 표현했 듯 spawn의 특별 케이스로 새로운 Node.JS 프로세스를 생성하는 함수입니다.
이때, 개별 V8 프로세스가 생성되고, 부모 프로세스와 IPC channel이 생성됩니다.[[3]]
자동으로 IPC channel을 생성해 주기 때문에 활용하기 쉬운 관계로, 저는 이 글에서 fork를 사용하였습니다.

### 예시
```javascript
import { fork } from "child_process";

const worker = fork(filePath);

worker.on("message", (message) => {
    // do something
});

worker.on("exit", (code) => {
    // do something
});
```

## cluster
### 기능
같은 포트를 공유하는 child process를 쉽게 만들어 주는 모듈입니다.[[4]]
child_process의 fork를 통해 자식 프로세스를 생성하고, primary process가 connection을 받아 round robin으로 뿌려주는 방식을 기본으로 동작합니다.

기본적으로는 networking을 목적으로 사용되나, worker process가 필요한 다른 목적으로 사용될 수 있습니다.


### 예시
```javascript
import cluster from "cluster";

if(cluster.isPrimary) {
    const worker = cluster.fork();
} else {
    console.log("hello, this is child process");

    process.exit(0);
}
```

## worker_thread
### 기능
multi thread를 지원하는 모듈입니다. CPU intansive한 작업을 처리하는데 좋다고 합니다. [[5]]
thread간 데이터의 통신은 ArrayBuffer 혹은 SharedArrayBuffer를 이용해 처리한다고 합니다.
message channel을 열어서 처리하는 식으로 스레드간 통신을 합니다.

실질적으로 어떻게 처리하는지는 내부를 까봐야 알 것 같습니다.

### 예시
```javascript
import {Worker, isMainThread, parentPort} from "worker_threads";
import path from "path";

if (isMainThread) {
    const worker = new Worker(path.resolve(".")); 
} else {
    parentPort.send("done");
}

```

# io intensive + cpu intensive일 때의 비교
## 실험 환경
제 19년식 intel macbook pro를 사용하였습니다. 쿼드코어로 활용가능한 hyper threading을 통해 프로그래밍 적으로 8개 코어를 사용 가능합니다.
RAM은 8GiB로, 이중 30%정도를 시스템에서 활용하고 있습니다.

유튜브 뮤직과 chrome, vscode, terminal이 떠있는 상태입니다.

## 기본 실험
### 실험 가설
1. cluster가 child_process를 사용하므로, cluster와 child_process는 성능상 유의미한 차이가 없을 것이다.
1. cpu intensive job에서는 cluster와 worker_threads의 유의미한 차이가 없을 것이다.
1. io intensive job에서는 cluster와 worker_threads의 유의미한 차이가 있을 것이다.

### 실험 설계
#### cpu intensive
1억번의 덧셈 연산

#### io intensive
100번의 파일 쓰기 연산

#### 측정
10번의 실행 값의 평균

#### 단위
ms 단위, 소숫점 1자리까지 유효숫자

### code

#### timer
```javascript
export async function measureAsyncFunctionExecutionTime(promiseData) {
  const start = new Date();
  await promiseData;
  const end = new Date();

  return end - start;
}
```

#### cpu intensive
1억번의 연산을 수행합니다.

```javascript
export function cpuIntensiveJob() {
  const HUNDRED_MILLION = 100000000;
  let sum = 0;

  for (let i = 0; i < HUNDRED_MILLION; i++) {
    sum += i;
  }

  return sum;
}
```

#### io intensive
1번의 읽기, 100번의 쓰기 작업을 수행합니다.
```javascript
export async function ioIntensiveJob(fileName) {
  try {
    const baseFiles = path.resolve("./base_file.txt");
    const data = fs.readFileSync(baseFiles, 'utf8');
  
    await Promise.allSettled(
      Array.from({ length: 100 }, (_, i) =>
        fs.promises.writeFile(path.resolve(`./output/${i}-${fileName}`), data)
      )
    );
  } catch {
    // ignored
  }
}
```


### 결과
| 작업 | child_process | cluster | worker_threads |
|-|-|-|-|
| cpu | 5736.7ms | 6864.1ms | 6273.8ms |
| io | 504.3ms | 438.8ms | 465.5ms |

#### 설명
1. cpu 작업의 경우, 최초 시도에서는 child_process가 가장 빨랐으나 반복된 시도에서 숫자가 +-1000ms의 차이를 보여 전부 유효 범위 안에 들어왔다고 볼 수 있습니다.
1. io 작업의 경우, 3작업 모두에서 약 500ms 이하로 빠른 속도를 보였습니다.

#### 해석
1. io, cpu intensive 작업에서 아래 조건일 때 성능이 비슷합니다.
    1. cpu: 1억번의 연산 100개
    1. io: 1개의 파일 읽고 100개의 파일 생성하는 연산을 100번

## 추가 실험
### 실험 가설
1. 1개 process만 사용하므로, cpu intensive job의 부하가 심해질 수록 worker_threads는 cluster와 child_process에 비해 느릴 것이다.
1. io intensive job의 부하가 심해질 수록 worker_threads는 cluster와 child_process에 비해 느릴 것이다.

### 실험 설계
| 실험 | 1 | 2 | 3 |
|-|-|-|-|
| cpu | 1억 | 3억 | 5억|
| io | 100 | 500 | 1000 |

#### cpu intensive 
```javascript
// 위 실험과 횟수 빼고 동일
export function cpuIntensiveJob(scale) {
  const THOUSAND_MILLION = 100000000;
  let sum = 0;

  for (let i = 0; i < scale * THOUSAND_MILLION; i++) {
    sum += i;
  }

  return sum;
}
```

#### io intensive
```javascript
// 읽기와 쓰기를 한 연산안에서 함
export async function ioIntensiveJob(fileName, times) {
  try {
    await Promise.allSettled(
      Array.from({ length: times }, (_, i) =>
      {
        const baseFiles = path.resolve("./base_file.txt");
        const data = fs.readFileSync(baseFiles, 'utf8');

        return fs.promises.writeFile(path.resolve(`./output/${i}-${fileName}`), data)
        }
      )
    );
  } catch {
      // ignored
  }
}
```


### 결과

#### cpu intensive
| 실험 | child_process | cluster | worker_threads |
|-|-|-|-|
| 1 | 5736.7 | 6864.1 | 6273.8 |
| 2 | 22540.5 | 15801.3 | 18005.9 |
| 3 | 33736 | 27795.7 | 28974.2 |

#### io intensive
| 실험 | child_process | cluster | worker_threads |
|-|-|-|-|
| 1 | 2829.6 | 1773.8 | 4029.3 |
| 2 | 14676.2 | 9233.3 | x |
| 3 | 28841.8 | 50447 | x |

#### 설명
- cpu작업에서 cluster가 child_process보다 성능이 좋습니다.
    - 구현 특성상 차이가 발생했을 가능성이 있습니다. (child_process는 한번에, cluster는 한번에 하나씩 돌림)
- io작업에서 작업의 개수가 커졌을때 child_process가 cluster보다 큰 상황이 발생했습니다.
    - 기타 시스템의 영향을 받았을 수 있습니다.
- worker_threads의 io작업은 `too many open files`[[6]]이슈가 발생하여 측정하지 못했습니다.

#### 해석
- cpu 작업의 경우, 5억개의 작업까지는 worker_threads, cluster, child_process의 차이가 없습니다.
- io 작업이 많아질 수록 cluster와 child_process에 비해 worker_threads의 비용이 더 큽니다.
- cluster와 child_process는 각 작업에서 유의미한 차이를 보이긴 하나, 환경이 고정되지 않아 추가해석을 달기에는 부적합하다.
- process를 spawn하고 IPC를 통해 통신하는 비용을 작은 규모에서는 무시할 수 없습니다.
    
# 결론
## child_process를 써야 겠군요
1. 병렬처리시 파일별로 로직을 분리하기 쉽다.
1. io와 cpu 작업이 어느정도는 필요하다.
1. 같은 포트를 사용할 필요는 없다.

## 이럴때는 이걸 써야겠군요
### worker_threads
- cpu intensive 작업이지만 각 작업의 연산이 5억개 미만이고, 100개 작업 정도가 필요할 경우

### cluster
- io작업도 있고, cpu 작업도 있는 경우
- 같은 포트를 사용해야하는 경우

### child_process
- io작업도 있고, cpu 작업도 있는 경우
- 같은 포트를 사용할 필요는 없는 경우

## reference
- 1. [child_process spawn vs fork][1]
- 2. [NodeJS Child_process의 spawn(), exec(), fork()][2]
- 3. [nodejs: child_process][3]
- 4. [nodejs: cluster][4]
- 5. [nodejs: worker_threads][5]
- 6. [LINUX Too many open files 에러 해결하는 방법][6]

[1]: https://stackoverflow.com/questions/17861362/node-js-child-process-difference-between-spawn-fork
[2]: https://one-armed-boy.tistory.com/entry/NodeJS-Childprocess%EC%9D%98-spawn-exec-fork
[3]: https://nodejs.org/api/child_process.html#child_processexeccommand-options-callback
[4]: https://nodejs.org/api/cluster.html
[5]: https://nodejs.org/api/worker_threads.html
[6]: https://pinetreeday.tistory.com/171
