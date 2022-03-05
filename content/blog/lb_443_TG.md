---
title: "로드밸런서 도메인에 따른 규칙 적용"
date: 2022-02-28T18:49:11+09:00
slug: ""
description: ""
keywords: ["Load balancer"]
draft: true
tags: ["AWS","Network","Load balancer"]
math: false
toc: false
---

[내부 통신 https 적용(도메인 등록 편)](https://josephmjjo.github.io/blog/inside_network_https/) 

위의 도메인 등록 편에 이어서 WAF LB에 규칙 적용 편이다.

WAF LB에 도메인 인증서를 바꾼 이후 한 가지 문제가 발생했다.

기존에 쓰고 있는 ALB는 다른 도메인 주소도 같이 쓰고 있었다.

LB는 구조상 타겟 그룹과 연결되는데 문제는 기존의 타겟 그룹들은 80으로 받는다.

즉 구조가 WAF LB에서 80으로 쏘는 거 하나 443으로 쏘는거 하나 이렇게 2갈래로 나눠야 한다.

마음 같아서는 WAF LB를 두개 만들고 하나는 이번 프로젝트의 도메인 전용, 다른 하나는 다른 도메인들 이렇게 하고 싶었는데 

시간상 LB도 만들어야 하고 보안팀에 새로 만든 LB를 등록 해야한다.

시간상 너무 촉박했다. 근데 예전에 헤더 호스트에 따라 타겟 그룹을 다르게 할 수 있다는 것을 기억 났다.

[Application Load Balancer를 사용하여 호스트 기반 라우팅 설정](https://aws.amazon.com/ko/premiumsupport/knowledge-center/elb-configure-host-based-routing-alb/)

저 로드밸런서에 들어올 때 주소가 특정 도메민이면 (예를 들어 www.example.com) 443으로 따로 만든 타겟 그룹으로 보내는 것이다.

![Untitled](/img/lb_443_TG/Untitled.png)

해당 규칙을 이용하면 호스트 헤더 값에 따라 지정한 타겟 그룹으로 가거나 아님 리다이렉트를 할 수 가 있다.