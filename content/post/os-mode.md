---
title: [OS] 운영체제 모드
date: 2022-07-05T20:31:56+09:00
draft: false

categories:
- os
tags:
- operating system
- mode
keywords:
- os
- operating system
- kernel mode
- user mode
---
# 운영체제의 모드와 시스템 콜
## 정의
### 시스템 콜
유저 프로그램이 커널의 서비스를 받기 위해 사용하는 인터페이스

### 커널 모드
모든 자원에 접근, 명령이 가능한 모드

### 유저 모드
유저가 접근 가능한 영역, 제한적인 자원 접근을 허용

## 필요성
### 보안
유저 프로그램은 다양한 목적에서 만들어 질 수 있습니다. 저장된 데이터를 마음대로 삭제하기도 하고, 웹캠을 마음대로 키고 끌 수도 있습니다. 이런 상황을 최대한 막기위해서 커널 모드와 유저 모드를 구분합니다.

## 구현
### modebit
유저 모드와 커널모드를 구분하는 플래그 값

---
### 참조
- [BlockDMast, [운영체제] 유저모드와 커널모드에 대해서](https://blockdmask.tistory.com/69)
