---
title: "AWS에서 aurora database의 Major 버전을 업그레이드하는 방법"
date: 2024-02-01T18:22:58+09:00
draft: false

categories:
- AWS
tags:
- AWS
- Aurora DB
keywords:
- upgrade
- MySQL
---

# TL;DR
- Aurora MySQL db를 업데이트하는 2가지 방법을 소개
- 2가지 방법 중 좋다고 생각한 방법과 이유를 소개

# 왜 와이
## Aurora DB 2점대의 depreacation이슈
### MySQL 5.7의 지원 종료와 함께

몇일 전이었습니다. MySQL 5.7버전이 정식 지원 종료된다는 소식을 들은게... 
이에 AWS에서는 [AuroraDB 2점대를 3점대로 업그레이드 하라는 안내](https://repost.aws/articles/ARWm1Gv0vJTIKCblhWhPXjWg/announcement-amazon-rds-for-mysql-5-7-will-reach-end-of-standard-support-on-february-29-2024)를 주었습니다. 그걸 이제서야 본 것이죠.
부랴 부랴 문서를 찾아 읽기 시작했습니다. 다행이도 문서는 매우 잘 정리가 되어있었고, 한글로 봐도 될 정도의 번역으로 만들어져 있었습니다.

이 글은 그 문서들을 읽고 내린 의견과 문서를 약간 정리한 내용을 소개하는 글입니다.
당연하게도 잘못된 지식을 바탕으로 내린 결정일 수 있으니, 너그러히 알려주시면 감사드리겠스니다.

업그레이드와 관련된 공식 문서는 [여기](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Updates.MajorVersionUpgrade.html)를 참고해 주세요.

# 정리한 내용
## 방법은 2가지가 있다.
### 클러스터 자체에서 업그레이드 하는 방법
- 장점
    - 추가 설정 없이 같은 인스턴스에서 업그레이드 가능
- 단점
    - 업그레이드 과정 중에 write instance가 내려가는 경우가 발생한다.
    - 업데이트 중간 실패시 롤백이 자동으로 되긴하나 다시 시도 해야한다.
    - 쓰기 업데이트가 많을 경우 업그레이드에 필요한 시간이 늘어나는 것을 고려해야 한다.
### blue/green 배포 방법
- 장점
    - 전환 자체는 설정에 따라 1분만에 이루어 질 수 있다.
    - 버전 업을 할 경우 발생할 수 있는 오류들을 green instance에서 미리 테스트 가능하다.
- 단점
    - write instance를 재부팅 해야 한다.
        - 클러스터 파라미터 그룹의 `binlog_format`의 설정을 변경해야 green환경으로 데이터가 복제된다.
        - 클러스터 파라미터 그룹의 변경사항을 적용하기 위해서는 재부팅이 필요하다.
        
    - 전환시에는 쓰기 작업이 방지되는 이슈가 있다.


# 내린 결론
## Blue Green 배포를 사용하자.
### 안전성의 측면
- 클러스터 자체에서 업그레이드 하는 방법
    1. write instance가 업데이트를 위해 종료됨
    1. 업데이트가 얼마나 걸리는지 사전에 알기 어려움
- blue/green 배포 방법
    1. 최대 전환 시간을 설정하여 실패 관리 가능
    1. 클러스터 파라미터 그룹 설정의 적용을 위해 db instance 재부팅이 필요

### 속도의 측면
- 클러스터 자체에서 업그레이드 하는 방법
    1. 시간과 관련하여 워딩이 나오지 않음, 문서상으로 파악하는데 한계가 있음
- blue/green 배포 방법
    1. 설정에 따라 1분 이내에도 전환이 가능하다는 설명 존재

## 의식 하면 좋을 점
### db instance 재부팅에 걸리는 시간
개발 환경 등에서 미리 한번 돌려보는 것이 좋습니다. 미리 돌려보지 않았다간 쓰기 지연이 발생할 수 있습니다.
### aurora 3 버전을 지원하는 db 버전 확인하기
[다음 문서](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html)를 참고하시어 현재 db 버전이 업그레이드 하고자하는 aurora 버전을 지원하는지 확인해보시기 바랍니다.

### MySQL 5.7과 8의 차이
잘 정리된 다른 문서를 보시면서 확인해 보시기 바랍니다. 저의 경우 [다음 문서](https://planetscale.com/blog/upgrading-to-mysql-8#before-you-upgrade)를 읽었는데요, 추후 정리해볼 예정입니다.

---
이상 제 의견, 정리한 짤막한 부분을 소개하는 글이었습니다.
읽어 주셔서 감사합니다.

