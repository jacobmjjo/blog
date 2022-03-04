---
title: "Azure에서 VPN 로그 보는 법"
date: 2022-02-23T01:45:51+09:00
slug: ""
description: ""
keywords: ["VPN"]
draft: true
tags: ["Cloud","Network"]
math: false
toc: false
---


아직도 VPN 연결이 안정적이지 않다. 

이제껏 매번 MS 엔지니어를 거쳐서 로그를 전달 받다보니 굉장히 느리다. 

그래서 이번에 로그 보는 법을 알려달라해서 알게 되었다.

경로는 모니터→ 진단 세팅→ vpn 검색→ 진단 설정 추가

![Untitled](/img/azure_vpn_review/Untitled.png)

그럼 모니터에서 로그를 들어가서 로그를 검색할 수 있는데 문제는 이게 쿼리문이다.

여러가지 시도 끝네 알아낸 쿼리 문이 바로 아래에 있는 쿼리문이다.

```sql
AzureDiagnostics  
| where Category == "IKEDiagnosticLog" 
| extend Message1=Message
| parse Message with * "Remote " RemoteIP ":" * "500: Local " LocalIP ":" * "500: " Message2
| extend Event = iif(Message has "SESSION_ID",Message2,Message1)
| project TimeGenerated, RemoteIP, LocalIP, Event
| where RemoteIP == "124.243.8.133"
| where TimeGenerated > todatetime('2022-02-24T05:00:00') and TimeGenerated < todatetime('2022-02-24T05:40:00')
| where Event has "[RECV Network Packet]"
| order by TimeGenerated
```

처음에 AzureDiagnostics만 할 경우에는 아래처럼 나온다.

![Untitled](/img/azure_vpn_review/Untitled%201.png)

AzureDiagnostics가 아마 진단 설정을 한 로그들을 보여주는 것이다.

여기서 키워드는 category와 message다 category를 무엇으로 하는가에 따라 IKEDiagnosticLog나 **TunnelDiagnosticLog로 설정할 수 있다.** 

다른 카테고리에 대한 조회 예시는 링크를 참고하면 된다.

[Troubleshooting Azure VPN Gateway using diagnostic logs](https://docs.microsoft.com/en-us/azure/vpn-gateway/troubleshoot-vpn-with-azure-diagnostics)

아무튼 조회를 하다보면 송수신이 번갈아 가면서 로그를 남겨야하는데 

![Untitled](/img/azure_vpn_review/Untitled%202.png)

send만 연속으로 보내는 경우가 있다. 연결이 잘 안됐을때의 로그 현상이다.

![Untitled](/img/azure_vpn_review/Untitled%203.png)

또한 Azure측에서는 QM packet이라는 것을 증거로 Azure에서 송신을 하고 있다라고 이야기 한다.

![Untitled](/img/azure_vpn_review/Untitled%204.png)

현재로써는 정확한 원인은 찾지 못했고 우선적으로 Ikev2로 변경한 상태이다.

 25일 새벽 1:35분인 지금 핑을 단 한 번도 끊긴적이 없는 것으로 보면 연결이 잘 된거 아닌가 싶다.

추후 에러가 더 이상 발생하지 않으면 내용을 총 정리해서 업로드를 하겠다.