---
title: "내부 통신 https 적용(도메인 등록 편)"
date: 2022-02-28T18:32:10+09:00
slug: ""
description: ""
keywords: ["HTTPS"]
draft: true
tags: ["AWS","Network","Azure","Security"]
math: false
toc: false
---

지금 하는 프로젝트에서 마지막 단계인 HTTPS 도메인 인증서 도입 단계만 앞두고 있다.

개발사에서 waf 넘어서도 https로 받아야 한다고 했으니 아키텍처가 복잡해졌다.

신규 도메인 인증서 AWS에 등록 → WAF LB 인증서 변경 → 이번에 사용하는 도메인만 443으로 전달 설정 → waf에 도메인 인증서 등록(복호화 후 암호화??) → Azure application gateway에 https 적용

여기서 내가 해야 할 작업이 신규 도메인 AWS 등록과 WAF LB 인증서 변경, 특정 도메인만 443 호출(가장 어려웠음), Azure GW에 등록이다.

여기서  Azure와 AWS의 도메인 등록 방법이 다른 것이 꽤나 문제가 되었다. 

AWS에 등록하는 도메인 인증서는 pem이고 Azure는 pfx와 cer이다. 

그리고 AWS는 도메인 인증서를 한 번만 등록하면 되는데 Azure는 Application gateway의 수신기 따로 라우팅 규칙 따로 등록을 해야 한다. 이마저도 수신기는 pfx 라우팅 규칙은 cer이다.

나중에 외부 도메인으로 application gateway 이용시 항상 확장자 2개를 준비해야 한다는 걸 기억해야 한다.

여기서 cer을 준비를 못해서 application gateway는 빼기로 했다.

그럼 아키텍처는 다시 아래 처럼 바뀌게 된다.

신규 도메인 인증서 AWS에 등록 → WAF LB 인증서 변경 → 프젝 도메인만 443으로 전달 설정 → waf에 도메인 인증서 등록(복호화 후 암호화??) → 호텔 was 서버 통신

먼저 도메인 인증서가 필요한 이유는 HTTPS를 적용하기 위해서다. 

HTTPS는  **HTTP** + **S**ecurity의 약자로 말그대로 HTTP에 보안을 입힌 기술이다.

통신 도중 해커에게 데이터를 읽히거나 수정당하는 것을 막는 기술이다.

여기서 중요하게 알아할 개념이 SSL 보안 방식으로 두 시스템간의 링크에서 암호화를 적용하는 방식이다. 

SSL은 여러가지 암호화 방식을 적용하는 것으로 알고 있는데 그 중 대표적인 것은 RSA 방식이다.

더 HTTPS와 SSL에 대해 더 자세한 내용은 아래 링크를 참고하면 된다.

[What is https](https://www.tutorialsteacher.com/https/what-is-https)

[How HTTPS works](https://www.youtube.com/watch?v=w0QbnxKRD0w)

한국어 설명

[154. [Security] SSL과 인증서 구조 이해하기 : CA (Certificate Authority) 를 중심으로](https://m.blog.naver.com/alice_k106/221468341565)

아무튼 신규 도메인 인증서를 aws 에 등록 하려면 인증서가 pem 파일 형식이어야 하고 그럼 필요한 파일이 3가지 인데 공개키, 비밀키, chain이다. 공개키가 있는 것을 보면 RSA인 것을 알 수 있는데 신기한 건 chain이 하나 더 있다는 것이다.

chain의 역할은 공개키 인프라(PKI)의 CA간의 신뢰를 형성하는 것이다.

여기서 신뢰는 역할 상속성(**hierarchical roles)이고 관계는 root CA, the intermediate CA, and the Secure Sockets Layer (SSL) certificates 이렇게 세가지이다.**

SSL 인증서 구성은 세가지로 

1. URL 인증서
2. 중개 인증서
3. root 인증서

아까 역할 상속성이라는 말을 썼는데 가장 높은 권한을 가진 역할이 바로 밑 역할에게 권한을 허용을 하면 그 밑 역할은 가장 위와 같은 권한을 가지고 그 밑 역할은 자기 보다 밑 역할에게 권한을 부여 할수 있게 해준다.

말 그대로 싸인을 사슬처럼 계속해서 해주면서 권한을 부여 할 수 있다는 것이다.

아래 그림을 보면 가장 위가 Root이고 가운데가 중개 인증서 최하단이 url 인증서다

![Untitled](/img/inside_network_https/Untitled.png)

AWS 도메인 등록할때 체인을 등록하지 않으면 root ca 권한을 url 인증서까지 도달 할 수 가 없다.

더 자세한 내용을 알 고 싶다면 아래의 링크를 참고하면 된다.

[SSL Certificate Chain | Your in-depth Guide - https.in Blog](https://www.https.in/blog/ssl-certificate-chain/#:~:text=What%20is%20SSL%20Certificate%20Chain,Sockets%20Layer%20(SSL)%20certificates)

무튼 공개키, 개인키, 체인 이렇게 3가지를 등록하면 아래의 그림 처럼 유형 가져옴으로 도메인 목록에 추가된다.

(도메인 중복으로 사용할 수 있을거라곤 예상 못했는데 다행이다.)

![Untitled](/img/inside_network_https/Untitled%201.png)

다음편은 url 주소에 따라 타겟 그룹을 다르게 하는 방식이다.