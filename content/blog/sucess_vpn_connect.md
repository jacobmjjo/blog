---
title: "온프레미스와 Azure의 VPN 성공!!!"
date: 2022-02-28T02:29:38+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","Network","Azure"]
math: false
toc: false
---


오랫동안 시달렸던 Azure와 온프레미스와의 연결 문제가 드디어 해결 되었다.

결론은 호환성이다. 

버전이 최신이어야 하고 Ikev버전은 2로 해야 한다. 더불어 traffic selector는 정책 기반이어야 한다.

위 조건은 microsoft에서 공식 Azure 사이트 cisco와 통신을 할때 샘플 조건이다.

[VPN Gateway에 Cisco ASA 디바이스를 연결하는 샘플 구성 - Azure VPN Gateway](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-3rdparty-device-config-cisco-asa)


해당 버전에서는 cisco asa버전에서 ikev2를 사용하려면 8.4이상이면 된다고 하는데 정작 온프레미스는 8.6인데도 ikev2로 연결을 시도하면 통신이 갑작스레 안되는 경우가 잦았다.

어쩔 수 없이 ikev1로 변경했지만 역시나 통신이 어려웠다. 

MS엔지니어가 권장하기로는 최신버전으로 업데이트를 하는 것이었다.

Ikev2가 8.6버전에서 안되는 이유는 정확하게는 찾지 못했고 일종의 버그로 추정하고 있다.

그래서 최신 버전인 9.12로 업그레이드 했지만 문제는 Ikev1에서도 간헐적으로 통신 불가 현상이 발생했다.

로그를 보니 서로 통신으로 하고 있는 것으로 확인 되었다.

로그 보는 방식은 아래 링크로 들어가면 된다.

Azure의 로그를 살펴보니 Qm packet을 계속해서 보내고 있다.

![Untitled](/img/sucess_vpn_connect/Untitled.png)

여기서 QM이란 Ikev1의 phase2에 해당하는 것으로 Phase1 에서 ISKMP으로 안전한 방식으로 보안 통신 채널을 설정했다면 그 안전한 보안 통신에서 알고리즘과 VPN을 통해서 트래픽을 보내는 것을 빠르게 허용하는 것이다. 

아래의 링크를 보면 Ikev1은 phase1에서 2가지 모드를 설정 할 수 있는데 Azure는 Ikev1에 대한 모드를  선택할 수 가 없다. Azure에서 Ikev1 보다는 ikev2를 권장하기에 자연스레 포탈에서 설정할 수 있는 부분을 지원하지 않은 것 같다.

[CCIE Security: IPSec VPN Overview (IKEv1) - Networking fun](http://www.network-node.com/blog/2017/7/24/ccie-security-ipsec-vpn-overview)

내가 생각하기에는 Ikev1가 Azure VPN에서는 phase1 부분에서 뭔가 안 맞는 부분이 있는 듯하다.

다행히도 Ikev2로 변경한 결과 문제가 해결 되었다. 

다음에는 왜 Ikev에는 정책기반으로 해야하며 Ikev1과 Ikev2의 차이점 그리고 VTI 모드에 대해 공부해 보겠다.