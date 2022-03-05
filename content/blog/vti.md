---
title: "VTI란 "
date: 2022-03-01T00:36:55+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Network","Security"]
math: false
toc: false
---


Cisco ASA와 Azure VPN 연결 중 만약 정책 기반으로도 실패 할 경우 할 수 있는 방법으로는 VTI를 설정하여 경로 기반으로 통신 할 수 있게 하는 방법이다.

우선 VTI가 뭔지 알아야 한다. 

VTI란 Virtual Tunnel Interface의 약자로 사용자들이 IPsec기반 VPN 을 구성할 수 있도록 하는 새로운 도구이다. 공유된 WAN을 통해 지정된 경로를 제공하고 새패킷 헤더로 캡슐화 해서 특정대상에 대한 전송을 보장하는데 지원한다. 

즉 공유된 WAN을 기반으로 경로가 제공되니 이 경로 안에서는 통신이 원할하게 가능하다는 것이다. 

VPN IPsec 할 수 있도록 하고 GRE 대안으로 많이 사용한다.(GER에 대한 이야기는 아래 한국어 설명이라고 써 있는 링크 참고) 

VTI의 큰 장점은 Multi 캐스트 기능이 가능하고 또한 구성도 간편하다.

정책 기반에서 안될 경우 경로 기반이 대안이 되는 이유는 이 멀티캐스트 때문인 것 같다. 정책기반에서 네트워크 조합이 매핑이 잘 안될 경우 차라리 와일드 카드를 써서 다 연결하면 될 것 같다는 의미에서 대안이 되는 것 같다.

전에 포스팅 했던 정책 기반과 경로 기반 내용을 보면 그림으로 무슨 차이가 있는지 알 수 있다.

[Azure VPN 정책기반과 경로 기반](https://josephmjjo.github.io/blog/vpn_policy_route/)

VTI에 대한 자세한 설명

[IPSec Virtual Tunnel Interface](https://www.cisco.com/en/US/docs/ios/12_3t/12_3t14/feature/guide/gtIPSctm.html#wp1027171)

한국어 설명

[GRE와 VTI](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ccw928&logNo=220610875688)

그럼 VTI를 설정하려면 Cisco 장비에서 설정을 해줘야 한다.

VTI 설정 방식은 아래 링크의 가이드 대로 하면된다.

Adsm을 이용한 vti 설정 방법 1

[Azure: Site-to-Site VPN with a Cisco ASA using ASDM](https://geekshangout.com/azure-site-to-site-vpn-with-a-cisco-asa-using-asdm/)

[Azure에 대한 ASA IPsec VTI 연결 구성](https://www.cisco.com/c/ko_kr/support/docs/security/adaptive-security-appliance-asa-software/214109-configure-asa-ipsec-vti-connection-to-az.html)

[샘플 구성: Cisco ASA 디바이스(IKEv2/BGP 아님)](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-3rdparty-device-config-cisco-asa)