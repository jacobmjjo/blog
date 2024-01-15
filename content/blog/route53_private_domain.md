---
title: "Route35 private domain"
date: 2022-11-21T16:10:06+09:00
slug: ""
description: ""
keywords: ["Private hosting","Route53"]
draft: true
tags: ["AWS","Network","Route53"]
math: false
toc: false
---

이번 프젝에서 어려웠던 부분 중 하나였다.

기존의 이미지 저장소인 ECR이 매 업로드때마다 통신비용이 단점이 있었다.(NAT를 통해서 나가는 비용)

그래서 빌드 이미지 업로드 관련해서는 비용을 절감하는 방법을 찾아보다가 cicd 서버안에 저장소를 만들고 거기에 이미지를 추가하는 방식으로 바꿔보았다.

현 프로젝트의 CICD 서버는 2대이다. 하나는 gitlab이고 또 다른 하나는 gtibuild이다.

2대로 나눈 이유는 만일 build와 deploy를 하나에 하게 되면 빌드중 부하로 인해 다른 서버의 배포작업을 못하기 때문이다. 그래서 build용 서버를 별도로 두었다.

아무튼 ECR대신 리포지토리 작업에 있어서 중요한 점은 HTTPS 통신 여부이다.

build 서버에서 빌드를 완료한 후 리포지토리에 보낼려면 https://리포지토리 domain으로 보내야 한다.

이 HTTPS를 사용하려면 몇가지 필요한 점이 있다.

1. SSL 인증서를 어디다 적용할 것인지
2. 도메인을 내부에서 어떻게 사용할 것인지
3. 다른 계정에 있는 VPC에 어떻게 리포지토리 도메인을 적용할 것인지

이렇게 3가지가 있다.

AWS에서는 1번이 큰문제가 되는데 이유는 SSL 인증서를 파일로 주지 않기 때문이다.

AWS의 리소스에만 적용이 되고 ec2안에 있는 솔루션에 SSL인증서를 적용하려면 별도로 파일이 필요하다.

외부에서 구매를 하려면 비용절감이라는 목적에 맞지 않아 다른 방법을 적용했다.

외부에서 SSL 인증서를 구매하지 않고도 적용을 하려면 결국 aws 리소스를 사용해야 한다.

그래서 AWS 리소스 중 ALB를 추가해서 리포지토리 도메인을 사용하기로 했다.

리포지토리 저장 공간은 gitlab 서버에 하기로 했었는데 다행히도 gitlab서버는 이미 로드밸런스가 연결되어 있는 상황이었다.

기존의 로드밸러스에서 특정 도메인만 리스너를 추가하면 된다.

근데 문제가 하나 더 생기는데 해당 도메인을 내부로만 사용하게끔 별도로 프라이빗 호스팅을 만들어야 한다.

이 프라이빗 호스팅이 이번주제의 핵심이다.

우선 route53의 워크플로우를 이해할 필요가 있다.

Rout53은 AWS에서 제공하는 DNS 웹서비스로 도메인을 입력하면 그와 매치된 네임서버, LB 등 기타 여러 리소스에 전달하는 기능이다.

각 도메인을 어떻게 호스팅 할것인지 매치하는 것으로 호스팅이라고 한다.

이 호스팅을 보통 퍼블릭으로 많이 사용한다. web서버 만들때 퍼블릭으로 사용할 도메인을 Route53을 통해서 구매하면 자동으로 도메인을 AWS 네임서버에 매치된다. 이 매치된 내용을 Route53 호스팅에서 확인이 가능하다.

route53 콘솔에 가보면 아래처럼 자동으로 네임 서버가 매치된 걸 확인 할 수 있다.

(여기서 NS 유형이 네임서버 유형이다. SOA 유형은 나중에 찾아 보겠다.)

![Untitled](/img/private_hosting/route53.png)

예시는 퍼블릭 호스팅이지만 프라이빗도 동일하게 자동으로 매치가 된다.

근데 이번 케이스는 조금 특이한 면이 있다.

와일드카드 도메인을 이미 퍼블릭으로 호스팅 했는데 특정 세부 도메인만 프라이빗 호스팅을 해야하는 입장이다.

예를 들어 \*.sample.io으로 퍼블릭 도메인을 만든 상황에서 registry.sample.io만 별도로 프라이빗으로 만들어야 한다.

만드는 방법은 어렵지 않지만 의외로 체크해야하는 부분들이 있다.

1. 도메인 정보의 프라이빗 호스팅 네임서버 추가
2. (optinal) 다른 vpc도 사용할 시 프라이빗 호스팅에 vpc 연결

1번은 내부에 HTTPS 통신을 위해 SSL 인증서를 추가 하려다가 알게 된 부분이다.

보통 ACM에서 SSL 인증서를 만들면 해당 도메인을 사용하는 Rout53 DNS 관리 호스팅 영역에 간단하게 등록할 수 있다

(아래 이미지 참조)

![Untitled](/img/private_hosting/certification.png)

하지만 이상하게도 프라이빗 호스팅은 SSL인증서 발급이 되지 않는다
발급되지 않는 문제 때문에 엄청 헤맸다

[[TIL 2/5] 텔넷(Telnet)명령을 통해 HTTP 요청하기](https://velog.io/@sms8377/TIL-25-%ED%85%94%EB%84%B7Telnet%EB%AA%85%EB%A0%B9%EC%9D%84-%ED%86%B5%ED%95%B4-HTTP-%EC%9A%94%EC%B2%AD%ED%95%98%EA%B8%B0)

[VPC의 DNS 속성](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-dns.html#vpc-dns-updating)

[고정 IP 주소에 대해 Network Load Balancer 뒤에 Application Load Balancer 등록](https://aws.amazon.com/ko/premiumsupport/knowledge-center/alb-static-ip/)

[트래픽을 웹 서버로 전달하기 위해 로드 밸런서가 사용하는 IP 주소 찾기](https://aws.amazon.com/ko/premiumsupport/knowledge-center/elb-find-load-balancer-IP/)
