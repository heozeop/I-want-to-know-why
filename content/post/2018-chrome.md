---
title: "2018 chrome browser 동작 글 읽고"
date: 2023-08-15T20:58:14+09:00
draft: false

categories:
- web
tags:
- fundamental
keywords:
- chrome
- 2018
---
각 소제목에 해당하는 글을 읽고 정리한 내용입니다.

## 1. chrome browser의 architecture

> https://developer.chrome.com/blog/inside-browser-part1/
> 

### browser가 선택하는 architecture

- tab별로 여러 process로 나눠서 실행하는 것
- 한 process가 다 처리하는거

### process 종류

- browser: ui, data fetching, ui,
- renderer: 그리기, service worker
- plugin: plugin
- gpu: 다른 processe들의 gpu task

### chrome은?

- browser process에 대해서 servicification이라는 개념을 도입한다.
    
    → computing 자원이 풍부한 환경이라고 판단되면 browser process의 동작들을 process 여러 개로 할당하고
    
    아니면 같은 하나의 process가 여러 동작을 하도록 설정하는 것
    
- [site isolation](https://developer.chrome.com/blog/site-isolation/)
    - site 별로 renderer process를 할당
    - same site policy, meltdown, spectre 등의 이슈를 대응하기 위해 도입이 필요하다고 판단

## 2. navigation시 동작

> https://developer.chrome.com/blog/inside-browser-part2/
> 

### 동작과정

1. user input
    1. browser process의 ui thread가 검색인지, url 입력인지 판단
2. 출발
    1. DNS, TLS 등을 network thread한테 시킴
    2. 301 같이 redirection 하는 거면 다시 2번 함
    3. tab에 loading 돔
3. read response
    1. url 검증
        1. ui thread가 현재 url과 domain이 다른 경우 검증 시행
            1. [CORB (cross origin read blocking)](https://www.chromium.org/Home/chromium-security/corb-for-developers/)
                1. MIME type가지고 정확한지 확인해서 일단 빈거 노출하게 함
            2. safe browsing
    2. content type을 보고 행동 생각
    3. network thread에서 정보를 ui thread로 줘서 해석
4. data의 content type이 HTML인 경우
    1. ui thread가 해당 site의 renderer process를 찾아 rendering 하라고 IPC날림
        1. 속도 때문에 data fetching할때 url가지고 먼저 찾긴 함
    2. 이때, 와리가리 button 및 주소창 update 됨
    3. commit 단계로 부름
5. rendering 완료시 (js 등 기타는 알빠 아님)
    1. renderer process가 다 됐다고 browser process에 던짐 → ui thread가 받아서 처리함
        1. 이때, 모든 Element의 onload event 발행이 끝난 상태임

### a button click 의 경우

- renderer process가 받음
- beforeunload 호출
    - navigation을 시작 전에 막는 거니, 사용에 유의
- domain 다르면, browser process에 navigation 바꾸라고 던짐
    - unload 등의 event 대응을 위해 상태 유지하고 대기
- renderer process 찾기 시작

### service woker

- service woker가 등록된 페이지라면, 외부 data fetching을 cache로 대체 가능
- navigation 발생시 network thread가 service worker scope로 look up 함
    - cache에 있으면 fetching 안함
    - cache 없으면 가져오는데, 일단 그냥 출발함 (navigation preloading)

## 3. renderer process

> https://developer.chrome.com/blog/inside-browser-part3/
> 

### renderer process?

- 실제 tab에 rendering 되는 요소들에 대한 관리를 맡는다.
- 구성
    - main thread
    - raster thread
    - compositor thread
    - worker threads

### rendering 단계

1. Parsing
    1. DOM을 만듦
    2. generous 함
    3. JS 등 sub source 만나면 block + parse 후 그림
        1. 최적화를 위해 preload scanner가 돔.
        2. async, defer
        3. preload
2. ADD Style
    1. DOM Node 마다 class를 계산해서 style을 적용.
    2. css 속성 보면 “계산됨(computed)”라고 있음.
3. Layout
    1. 물리 좌표 계산. (bounding box size 등)
    2. display none은 layout tree에 포함되지 않음.
4. Paint
    1. z index등을 사용해 element의 order를 계산
5. composition / rastering
    1. rastering
        1. dom node, style, layout, paint를 모아서 pixel로 만드는 행위
    2. 진짜 그리는 단계
        1. 초기: 스크롤 되면 스크롤 된만큼 그림
        2. 지금: 일단 layer 대로 rasterizing해 두고, 필요한 frame을 그림
    3. composition 선택 이유
        1. rastering을 한번만 함
        2. animation은 layer를 옮기는 것으로 가능함
        3. main thread에서 안함
            1. compositor thread가 tile로 page 나눔
                1. zoom in 고려해서 여러개 만듦
            2. raster thread가 tile을 gpu thread한테 던짐
                1. gpu memory에 pixel data 기록
            3. reastering 되면, compositor thread가 정보 모음
                1. draw quads: 그릴 tile의 정보 (memory, position 등)
                2. compositor frame: frame 구성하는 draw quads 모음
    4. composition 완료
        1. browser process로 IPC frame 제출
        2. ui thread가 gpu에 그리라고 던짐
            1. ui thread에서 주소 창 등 update가 필요한 부분에 대한 compositor frame을 더할 수 있음

### layer 만들고 싶다.

will-change 써라 (조심해서)

## 4. renderer process의 동작

> https://developer.chrome.com/blog/inside-browser-part4/
> 

### event 발생과 동작

1. 이벤트 발생 → browser process 감지 → renderer process에 던져 줌
2. main thread가 hit test라는 걸 compositor thread에 시킴
    1. hit test는 rendering 과정에서 생성한 paint records를 바탕으로 event target 찾는 과정
    2. 찾다가 event 나오면 main thread에 실행시키라고 해야됨 
    → 없으면 main thread 안쓰고 composition frame 생성 가능

### Non-fast scrollable

event handler가 달린 component를 composition 할 때, 해당 부분에서 event가 발생하면 main thread를 호출 해야한다고 compositor thread가 set하는 flag

### event handler의 passive option

일단 composition을 하라고 browser에 알리는 역할을 한다.

### event coalesce

touchmove 등의 연속 된 event나 moucemove등의 이벤트가 animatedFrame안에서 한번만 일어날 수 있도록 합치는 작업을 말한다. 그림 등의 작업에서 합쳐진 정보가 필요하다면, event객체의 getCoalescedEvents 함수를 쓰면 된다.