---
title: "Writing Secure Code"
date: 2022-11-13T11:42:43+09:00
draft: true
---

### 읽는 이유
1. Oreilly 1년치를 promotion 전에 질렀습니다. (약 72만원)
1. 되게만 하면 되는 코드를 짜는 "코드 몽키"에서 탈출할 것입니다.

### 읽는 방법
1. 한번 쭉 읽고, 내용을 영어로 정리합니다.
1. 한글로 내용을 축약해 정리합니다.
1. 한글 내용을 요약해 정리합니다.

> 이번 글은 영어로 정리한 초본입니다.

# 3. Security Principles to Live by
## SD^3
The author suggested sd^3, which stands for Security by Design, Default and in Deployment.
### To secure by design
means musts to acheive security depends on the develop chepters
- try to maintain simplicity of design and code as much as possible.
- do penestration test for product before shipping the product to the client.
    - test that break down the product
- set profer security target model for later testing
- do integration test

### To secure by default
means secure enough in the first place
- do not include all features in default.
    - give easy way to enable not default features
- give least privileges to the application
    - open when only needed
- apply appropriate resoruce protections

### To secure in deployment
means it's possible to maintain the security after the user installed your application
- offer a way to maintain security.
- make easy to update security patches
- provide information for users who want to use in secure manner how to do that.

## security principles to maintain security

### principles that should aware of
- Learn from mistakes
- Minimize your attack surface
  - Number of open sockets (TCP and UDP)
  - Number of open named pipes
  - Number of open remote procedure call (RPC) endpoints
  - Number of services
  - Number of services running by default
  - Number of services running in elevated privileges
  - Number of ISAPI filters and applications
  - Number of dynamic-content Web pages
  - Number of accounts you add to an administrator’s group
  - Number of files, directories, and registry keys with weak access control lists (ACLs)
- Use defense in depth
  - always think your environment is not safe enough to protect the application
- Use least privilege
  - even in your computer
  - use minimal privileges to your application
  - check if the application should be run with administer's account and change it.
  - seperate privileges to contain lowest privileges.
- Employ secure defaults
  - make default to be secure
- Remember that backward compatibility will always give you grief
- Assume external systems are insecure
- Plan on failure
- Fail to a secure mode
    - if you want to check whether or not the error occur, check no error rather than check the type of error
    - [The Protection of Information in Computer Systems](http://web.mit.edu/Saltzer/www/publications/protection/)
- Remember that security features != secure features
- Never depend on security through obscurity alone
- Don’t mix code and data
- Fix security issues correctly
## felt
- care about security
- stick to security as default
- check error secualiy
    - don't forget that fail to secure mode is important.


