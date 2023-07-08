---
title: "[NestJS] 스웨거에 BasicAuth 더하기, feat fastify,GCP"
date: 2023-07-06T21:36:00+09:00
draft: false

categories:
- NestJS
tags:
- nest
- back-end
- web
keywords:
- nestJS
- basic auth
- GCP Cloud Run
- fastify
---

# 스웨거
흔히 스웨거(Swagger)라고 부르는 OpenAPI를 사용할때, 저는 누가 어떻게 보느냐를 고민합니다.
직접 인증을 구현한 경우는 로그인을 시키고, AWS를 사용하는 경우 ip로 접근을 불가하게 만들기도 했습니다. 하지만 GCP의 CloudRun을 이용하는 경우, 구체적인 자원에 접근하기 어려워 Ip로 접근을 차단하는 것은 한계가 있었습니다.

그래서 떠올린게 basic auth였습니다. 물론 brute-force 공격에는 뚫리기야 하겠지만, 비용 대비 얻는게 크다고 생각했습니다. 제가 원하는 건 아무나 보지 못하는 것이고, 보안 관리는 기본적으로 구글에서 해주기 때문에 제 목적에 딱 알맞는 것이었죠.

# basic auth?
## 정의
[wikipedia](https://en.wikipedia.org/wiki/Basic_access_authentication)에 따르면, basic auth는 HTTP 1.0이 발표된 1996년에 정의된 방식으로, 아이디와 패스워드를 브라우저에서 받아 로그인을 처리하는 방식입니다. 아이디와 패스워드는 `:`으로 조인 된 뒤, Base64로 암호화 돼 `WWW-Authenticate` 헤더에 포함되어 쿼리 됩니다. 해시등 추가적인 암호화는 없기 때문에 보안을 위해서 https와 함께 쓰이는게 권장된다고 합니다.

# nest swagger에서
## basic auth
간단히 코드부터 보여드리면 아래와 같습니다. 저는 fastify를 사용해서 인스턴스를 받아와서 처리했습니다. 라이브러리는 `@fastify/basic-auth`를 사용했습니다. 물론 간단히 header를 직접 해석해서 쓰셔도 될겁니다.

```typescript
  const fastifyInstance = await app.getHttpAdapter().getInstance();
  await fastifyInstance.register(fastifyBasicAuth, {
    validate: basicAuthValidator,
    authenticate: { realm: "not public" },
  });

  fastifyInstance.addHook("onRequest", (req, res, done) => {
    if (req.raw.url.startsWith(`/doc`)) {
      fastifyInstance.basicAuth(req, res, done);
    } else {
      done();
    }
  });
```
# 정리
오늘을 짧게 `basic auth`와 `fastify` + `nest`에서 설정방법을 보았습니다. 

한글을 사용하는 방법등은 WIP로 이 글에 추가해 보겠습니다.

읽어주셔서 감사합니다.


