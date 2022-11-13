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

# 4. Theat modeling 
>  developers to trace every byte of data as it flows through their code and to question all assumptions about the data. For testers, it was data mutation, which we’ll cover in Chapter 19. And for designers it was analyzing threats. 

- developers need to make valid assumetions, designers need to analyze and testers need to think to breaking the application via data.

## secure design through threat modeling
- makeing thread modeling
    - good for
        1. understand application
        1. find bugs 
        1. help newbies to understand the application
        1. cross check for other application's threat
    - bad because
        1. it definitely took time
            1. comparably small though
### creating threat model
1. Assemble the threat-modeling team.
    1. lead by threat finder who know how the hacker hacks.
    1. focusing on finding not solving
1. Decompose the application.
    1. data flow diagram helps to work around creating threat modeling
        1. it doesn't mean that DFD is the best, UML also do work. but UML focus on "control" on the other hand the DFD focus on "data"
        1. avoid rabit hole, your focus should be finding threats on your system not desgining the appalication.
1. Determine the threats to the system.
    1. DNS spoofing, DNS poisoning
    1. there is category of security (STRIDE)
        > think cause and effect in the same time.
        1. spoofing
            1. fooling others
        1. tempering with data
            1. malicious modification
        1. Repudiation
            1. nonrepudiation - ability to check whether it's real
            1. refuse some authorization or facts
        1. information disclosure
            1. not granted access related
        1. Deny of service
            1. DDoS
        1. Elevation of priviliges
    1. [Octave](https://resources.sei.cmu.edu/library/asset-view.cfm?assetID=8419)
    1. threat tree
        1. there is "falut tree" that used to find threats on hardware industry. threat tree is software system version of fault tree.
        1. threat needs to stick to direct target of threat targets which identified on application decompose step.
        1. one tree for on reason on each threat target.
1. Rank the threats by decreasing risk.
    1. just measure the risk with consistence and realistic manner.
    1. one example is criticality(1~10) * possibility(1~10)
    1. DREAD
        1. Damage potential
        1. Reproducibility: How easy is it to get a potential attack to work?
        1. Exploitability: How much effort and expertise is required to mount an attack?
        1. Affected users
        1. Discoverability: assume 10
1. Choose how to respond to the threats.
1. Choose techniques to mitigate the threats.
1. Choose the appropriate technologies for the identified techniques. 
# security technique
## list of look for
### authentication
1. Basic authentication
1. Digest authentication
1. Forms-based authentication
1. Passport authentication
1. Windows authentication
1. NT LAN Manager (NTLM) authentication
    1. mitigates "man-in-the-middle" attack
1. Kerberos v5 authentication
    1. authenticate both direction => client <-> server
1. X.509 certificate authentication
    1. SSL/TLS
1. Internet Protocol Security (IPSec)
    1. data privacy and integrity
1. RADIUS
    1. Remote Authentication Dial-In User Service
### authrization
1. Access control lists (ACLs)
1. Privileges
1. IP restrictions
1. Server-specific permissions
### Tamper-Resistant and Privacy-Enhanced Technologies
1. SSL/TLS
1. IPSec
1. DCOM and RPC
1. EFS
    1. encripting file system
### Protect Secrets, or Better Yet, Don’t Store Secrets
if no secret, no need to protect
### Encryption, Hashes, MACs, and Digital Signatures
encrypt data with some sort of secret key to verify sender on receiver

### Auditing
- logging
### Filtering, Throttling, and Quality of Service
- filtering
    - filter some ip packets 
- throttling
    - limit on connection to anonymous request
    - give more for authenticated request
- Quality of Service
    - choose prefered traffic and give more resources

## thoughts 
I need experience to do modeling to understand the core concept to threat modeling like DFD, STRIDE, DREAD and threat tree.
