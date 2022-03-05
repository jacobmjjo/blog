---
title: "Azure VPN 정책기반과 경로 기반"
date: 2022-03-01T22:05:44+09:00
slug: ""
description: ""
keywords: ["VPN"]
draft: true
tags: ["Cloud","Network","Azure"]
math: false
toc: false
---

온프레미스와 VPN연결을 할때 정책 기반과 경로 기반을 고를 수 있다.(IKev2일 경우)

정책기반은 두 네트워크의 접두사 조합을 사용해서 트랙픽의 암호화/복호화를 IPsec 터널을 통해 하는 방법을 정의 한다고 한다.

경로 기반은 VPN 디바이스는 임의(와일드 카드) 트래픽 선택기를 사용하며 라우팅/전달 테이블이 서로 다른 IPsec 터널로 트래픽을 전달하도록 하는 것이라고 한다.

즉 정책 기반은 각각의 네트워크 접두사를 기준으로 연결을 했기에 설정된 접두사 조합으로만 연결이 가능하다. 예를 들어 10.1 ↔ 10.3 과 10.1 ↔ 10.2 두개로 트래픽 선택기를 설정했다면 10.3↔10.2는 안된다는 것이다.

반면에 경로기반은 와일드 카드를 트랙픽 선택기로 사용 했기 때문에 정책기반과는 달리 트래픽을 넘나들 수 있다.

이해 하기 쉽게 Azure 공식 사이트에서는 이미지를 제공해준다.

Azure의 가이들 링크: 

[여러 온-프레미스 정책 기반 VPN 디바이스에 VPN 게이트웨이 연결 - Azure VPN Gateway](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)

![출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)](/img/vpn_policy_route/Untitled.png)

출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)

![출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)](/img/vpn_policy_route/Untitled%201.png)

출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)

정책 기반일때 다른 트래픽으로 안되는 이유

![출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)](/img/vpn_policy_route/Untitled%202.png)

출처: [https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-connect-multiple-policybased-rm-ps)

위에 링크한 공식 가이드에서는 정책 기반 설정을 powershell로 하는 법만 알려준다. 

정책 기반 설정은 포탈에서도 가능하다.

![Untitled](/img/vpn_policy_route/Untitled%203.png)

만약 Cisco ASA 온프레미스와의 VPN 연결에서 정책기반도 되지 않을 경우에는 경로 기반으로 해야 하는데 그러기 위해서는 CIsco ASA에 VTI 방식을 설정해야 한다.