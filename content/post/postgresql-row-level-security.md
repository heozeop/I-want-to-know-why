---
title: "Postgresql Row Level Security(로우 레벨 시큐리티)"
date: 2023-12-26T19:50:50+09:00
draft: false

categories:
- postgreSQL
tags:
- postgreSQL
- Row Level Security
keywords:
- RLS
- Row Level Security
- postgreSQL Row Level Security
- 로우 레벨 시큐리티
---

ref: https://www.postgresql.org/docs/current/ddl-rowsecurity.html

---

# Row Level Security

## 메타

### 정의

- boolean을 리턴하는 query를 테이블의 row에 접근할 때 호출해 접근 권한을 확인하는 보안 방법
- 정의된 [policy](https://www.postgresql.org/docs/current/sql-createpolicy.html)를 따름.

### 동작

1. optimizer의 optimization등의 [leak proof fucntion](https://www.postgresql.org/docs/current/sql-createfunction.html) 동작
2. RLS 확인
3. 주어진 query 수행

### 활성화

- `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`
    - table owner등 bypassRLS 조건인 경우, RLS 확인을 강제하고 싶은 경우 아래 command 사용
    - `ALTER TABLE ... FORCE ROW LEVEL SECURITY`
- 활성화 여부는 policy 삭제와 관계 없음
    - table의 RLS가 disabled라도 policy는 편집 가능

### 고려 사항

- performance
    - row 접근시 마다 확인하기 때문에 영향을 줄 수 있다.
    - [indexing, sub query 등으로 속도 개선할 수 있다.](https://github.com/orgs/supabase/discussions/14576)
- race condition
    - view 권한의 업데이트 과정에서 오류가 발생할 수 있다.
        1. a, b 유저 존재
        2. a의 권한레벨은 2, b 의 권한레벨이 4
        3. b가 a의 권한레벨을 1로 낮추고 권한 레벨 2의 정보를 생성한다.
            1. 이와 동시에 a가 조회할 경우 b가 작성한 내용을 열람할 가능성이 있다.
    - 해결 방안
        - update시에 ACCESS EXCLUSIVE LOCK 걸기 등

## Policy Syntax 분석

### syntax

```sql

CREATE POLICY name ON table_name
    [ AS { PERMISSIVE | RESTRICTIVE } ]
    [ FOR { ALL | SELECT | INSERT | UPDATE | DELETE } ]
    [ TO { role_name | PUBLIC | CURRENT_ROLE | CURRENT_USER | SESSION_USER } [, ...] ]
    [ USING ( using_expression ) ]
    [ WITH CHECK ( check_expression ) ]
```

### as [Permissive vs Restrictive]

- Permissive (default)
    - OR 조건으로 조건 확인
- Restrictive
    - AND 조건으로 조건 확인

### to [Role]

- authenticated 등 대상 user 조건의 확인
- public으로 할 경우, 모든 유저 대상으로 query 실행
    - ` RLS 먼저 실행 후 모든 유저 대상으로 실행

### Using === WITH CHECK

- 동일한 역할을 하는 이름
- boolean 값을 리턴해 주면 됨.
    - false가 아니면 true로 처리하는 듯 함.


