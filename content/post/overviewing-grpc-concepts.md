---
title: "[개발/RPC] 무에서 gRPC개념 쌓기"
date: 2022-10-16T16:13:21+09:00
draft: false

categories:
- gRPC
tags:
- RPC
- gRPC
- protobuf
keywords:
- RPC
- gRPC
- protobuf
---

> !주의! 뜬구름 잡는 소리를 할 수 있습니다. 댓글 주시면 반영할 수 있도록 하겠습니다.

# RPC가 뭐여

## 정의

Remote Procedure Call의 약자로, 외부 procedure를 실행하는 것

## 어서 도냐

grpc가 http2를 쓴댜 

→ UDP를 4계층으로 쓰고, 그 위에서 돌아가는 걸거여 

→ 그럼 application 계층인건가

## 장점

1. IPC 등의 작업 없이 일단 실행시키는 것 가능
2. 서로 다른 언어에서 rpc 통신 방법만 맞추면, 통신 가능

## 단점

1. 보안 문제가 있을 수 있음
2. 응답이 비동기임

# proto buffer는 또 뭐여

## 정의

google에서 만든 rpc 통신 규약으로, 강력한 message의 정의를 통해서 일관된 데이터 구조로 소통 가능

## 다른거랑 뭐가 달라

> [Schema evolution in Avro, Protocol Buffers and Thrift](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)

### 1. Avro
    1. schema를 version으로 관리하고, 딱 맞아야 함
    2. 예전 pub/sub message 만들때, console에서 만드는 기본값으로 존재했었음 (물론 나중에 protobuf 추가됨)
    3. 데이터에 주석을 포함한 모든 정보를 encoding 하므로, schema만 정의되면 모든 정보 가져올 수 있음
### 2. Thrift
    1. RPC framework임
    2. 2개의 JSON encoding, 3개의 byte encoding 방법을 지원함
        1. byte encoding 중 한개(DenseProtocol)는 c++에서만 됨
    3. 기본적으로는 proto buffer랑 표현방식은 같음
        1. verbose 한게 기본값임
        2. 끝에 00으로 끝나야됨
        3. 짧게하면 짧아지긴 하는데, protobuf 보단 긴 경우가 있음 (많은지는 모르것음 ← 생각)

## IDL을 byte로 바꿀때

1. number taging을 한다.
2. 첫 byte에 5개 bit를 number tag에, 뒤에 3개 bit를 type 으로 관리한다.
    1. type은 value의 타입을 말한다.
    2. 타입 앞에 repeated, optional, required가 있는데 protoBuf 3에서 required가 빠진 것 같기도 하다.
3. type별로, 뒤에 이어질 길이를 1~2byte에 나타낸다.
4. list타입은 별도로 존재하지 않아, 같은 tag로 여러개 쓰도록해서 list를 표현한다.

## proto 3

> [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3)
> 
### 기본값이 진짜 그 값인지, 없는건지 구분할 수 없다.
  boolean으로 상태 표시할 때, 키고 끄는거 2개 써야할 듯

### 타입 관련 주의
    1. string이랑 bytes는 repeated랑 기본적으로 같은데, numerical value는 아님
        1. repeated numerical value는 packed type으로 encoding 됨
    2. reserved로 삭제시 tag 사용 방지 가능
        1. 중복 안됨
        2. 19000 - 19999는 FieldDescriptor로 예약됨.
    3. enum
        1.  0 존재해야 됨
            1. 기본값
            2. proto 2 호환
        2. enum은 allow alias option이 true 면 중복 선언 가능
        3. 언어 따라 compile value가 다르니 주의
    4. any는 있고, byte로 컴파일 되는데 다는 안되어 있음 ← 사용시 주의
    5. oneof
        1. map이랑 repeated filed엔 적용 하믄 안됨
        2. 나머지 다 clear한다는데, default 적용한다는 거 같음
    6. map
        1. key는 floating point, enum, byte 빼고 다됨
        2. value는 map빼고 다됨
        3. key로 정렬됨
        4. repeated 안됨
    7. package
        1. namespace 같은거
    8. json to proto buf
        1. type by type으로 변환

### 추가 사항

- optimizing 할때 보라고 한 encoding 방법
    - [https://developers.google.com/protocol-buffers/docs/encoding](https://developers.google.com/protocol-buffers/docs/encoding)
- style 가이드
    - [https://developers.google.com/protocol-buffers/docs/style](https://developers.google.com/protocol-buffers/docs/style)
    - plugin 이 있지 않을까 ㅎㅎㅎㅎㅎㅎㅎㅎㅎㅎ

## gRPC 개념

### 통신 방법

1. unary
2. client stream
3. server stream
4. bidirectional stream

### 실행

동기와 비동기 모두 지원 (언어마다 사용법 다름)

### dead line / timeout

통신시 설정 가능

### 서로 끝나는게 다를 수 있음

server는 성공인데 client가 timeout 날 수 있음

→ 4개의 use case가 통신마다 존재하는 것

### metadata

server와 client간에 보낼 message type sync를 맞추기 위한 key value 형식의 정보

key와 value는 string이 기본 타입으로, value는 binary 정보를 가질 수 있음

- key 에 대한 제약
    - grpc-로 시작하면 안됨
    - binary의 정보의 경우, -bin으로 끝남
    - case sensitive 하지 않음

### channel

server에서 client 연결하라고 열어둔 port, 보통 100개인가 1000개라고 함 - 확인 필요

### proto buf와의 관계

gRPC가 기본 설정으로 protobuf를 씀 → json으로도 가능하긴 함

## 인증

> [https://grpc.io/docs/guides/auth/](https://grpc.io/docs/guides/auth/)

### 지원하는 거

1. ssl/tls
2. alts
3. token ← google
    1. 일반적으로 1번이랑 같이 써야한다고 함
        1. 그럼에도 불구하고 굳이 썼다는건 이거 써보라는 거 같긴함
    2. google 쓸때만 써야함 ← 도둑 맞으면 악용 될 수 있음

### 만약 따로 auth용 API를 쓴다면,

custom header를 이용하거나 MetadataCredentialsPlugin을 쓰는 방법을 생각할 수 있다. 하지만 그렇게 했을때, user정보를 위해 기존 service를 계속 호출 해야하는 문제가 발생할 수 있다.

## 에러
써봐야 알 것 같다.
