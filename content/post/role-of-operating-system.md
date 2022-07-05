---
title: "운영체제란?"
date: 2022-07-03T21:17:44+09:00
description: 운영체제의 정의와 역할을 알아봅니다.
draft: false

categories:
- os
tags:
- operating system
- role
keywords:
- os
- operating system
---

# 운영체제
## 정의
하드웨어, 시스템 리소를 제어하고 프로그램에 대한 일반적인 서비스를 지원하는 시스템 소프트웨어다.<sup>[1]</sup>

### 구조
드라이버 > 커널 > 시스템 콜 > 사용자 인터페이스

### 목적<sup>[1]</sup>
- 사용자에게 컴퓨터 프로그램을 쉽고 효율적으로 실행할 수 있는 환경을 제공한다.
- 컴퓨터 시스템 하드웨어 및 소프트웨어 자원을 여러 사용자 간에 효율적으로 할당, 관리, 보호한다.
- 제어 프로그램으로서 사용자 프로그램의 오류나 잘못된 자원 사용을 감시하고, 입출력 장치 등의 자원에 대한 연산과 제어를 관리

## 역할<sub>[2]</sub>
1. CPU 스케줄링과 프로세스 관리
    - CPU 자원의 할당, 프로세스의 생성/삭제/자원 할당/반환을 관리합니다.
2. 저장장치 관리
    - 메모리: 메모리의 할당, 사용 방법 등을 관리합니다.
    - 디스크: 파일을 어떻게 저장할 것인지 관리합니다.
3. I/O 디바이스 관리
    - 외부 입/출력 장치와 어떻게 소통할지 관리합니다.
4. 사용자 관리
    - 사용자간 접근 권한, 자원의 활용등을 관리합니다.

### 운영체제는 저장장치를 어떻게 관리하나요?
- 시스템 콜을 이용해 저장장치에 대한 처리를 진행합니다. 이를 통해  컴퓨팅 자원에 대한 접근을 제어할 수 있습니다.
- 가상 메모리를 사용하여 사용자 프로그램이 실재 자원을 고려하지 않고, 일관된 환경에서 동작할 수 있다는 가정을 제공합니다.

### 우리는 어떻게 입출력 장치를 사용하나요? (인터럽트 구조)
- 입력<sub>[3]</sub>
    1. 입력이 일어나면 입력 장치 앞에 있는 디바이스 컨트롤러가 인터럽트를 발생시킵니다.
    2. CPU는 현재 작업 상태를 저장하고, 인터럽트 처리를 위한 루틴을 찾아 실행합니다.
- 출력<sub>[4]</sub>
    1. 출력이 일어나야 하는 출력 장치 앞에 있는 디바이스 컨트롤러가 현재 데이터에 출력이 가능하다고 인터럽트를 겁니다.
    2. CPU는 지정된 I/O 레지스터에 출력할 데이터를 출력합니다.

---
### 참조
- 1: [위키피디아, 운영체제](https://ko.wikipedia.org/wiki/%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C)
- 2: [주홍철, 면접을 위한 CS 전공지식 노트](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=292815727)
- 3: [용감한 형제들, 운영체제](https://github.com/brave-people/brave-tech-interview/blob/main/contents/os.md)
- 4: [피그브라더, [Chapter 8] IO - 입력 및 출력 장치](https://it-eldorado.tistory.com/24)

[1]: https://ko.wikipedia.org/wiki/%EC%9A%B4%EC%98%81_%EC%B2%B4%EC%A0%9C
[2]: https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=292815727
[3]: https://github.com/brave-people/brave-tech-interview/blob/main/contents/os.md
[4]: https://it-eldorado.tistory.com/24