---
title: "traceroute와 ping 그리고 netstat"
date: 2022-02-23T17:49:59+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","Network"]
math: false
toc: false
---



네크워크 문제시 원인 파악을 위해 사용하는 도구를 살펴보다가 좋은 블로그 글이 있어 번역 및 요약을 했다

[Ping, traceroute, and netstat: The network troubleshooting trifecta](https://www.redhat.com/sysadmin/ping-traceroute-netstat)

제목에서 언급한 세가지 툴은 보통 네트워크 문제가 생길때 통신이 잘되고 있는지 아닌지 많이 사용한다. 

# Ping

각각 차이가 있는데 먼저 ping은 굉장히 많이 알려져 있는데 쉽게 설명하면 “님 거기 있나요?” 이 메시지를 원격 호스트에 보내는 거다. 만약 있으면 “네 있어요”로 오고 없으면 타임아웃이 뜬다. 

프로토콜은 ICMP로 포트 번호는 없다고 한다. 약자는 Internet control Message Protocol이고 말그대로 프로토콜 에러 와 통신이 가지 않을 경우 확인을 위해 만들어 진거다.

근데 찾아보니 두가지 타입이 있다네 하나는 type8(Echo Request) 또 다른 하나는 type0(Echo Reply)이다. 하나는 물어보는 거고 다른 하나는 답해주는 거다.

![출처: [https://www.geeksforgeeks.org/ping-in-c/](https://www.geeksforgeeks.org/ping-in-c/)](/img/traceroute_ping_netstat/Untitled.png)

출처: [https://www.geeksforgeeks.org/ping-in-c/](https://www.geeksforgeeks.org/ping-in-c/)

여기서 클라우드를 사용한다면 중요한 건 응답 해야할 서버 즉 receiver가 icmp 방화벽을 오픈 해야 한다.

아래처럼 하면 된다.

![Untitled](/img/traceroute_ping_netstat/Untitled1.png)

물론 sender도 아웃바운드에 icmp를 적용해야한다.

# **Trace route**

Trace route는 출발지 부터 도착지까지의 경로를 확인 시켜준다. 

경로를 따라가면서 홉의 IP와 대기 시간도 확인이 가능하다. 그럼 각각의 홉의 정보를 어떻게 확인이 가능할까?

Traceroute는 OS 타입에 따라 UDP 또는 ICMP를 사용한다. *nix system은 udp를 사용하고 포트번호는 33434이다. 윈도우는 icmp다 

traceroute는 사용 중인 프로토콜/포트에는 응답하지 않게해서 차단도 가능하다.(정확하게 어떻게 하는지는 아직 모른다...)

![출처: [https://support.maxcdn.com/hc/en-us/articles/360036558472](https://support.maxcdn.com/hc/en-us/articles/360036558472)](/img/traceroute_ping_netstat/Untitled%202.png)

출처: [https://support.maxcdn.com/hc/en-us/articles/360036558472](https://support.maxcdn.com/hc/en-us/articles/360036558472)

traceroute를 실행할 때  도착지로 packet도 같이 보낸다. 그 패킷에는 time to live(TTL)을 장착하고 보내는데 이 행위가  출발지로 ICMP 타임 종료 메세지를 반환하기 전에 얼마나 많은 hop에서 패킷으로 허용했는지가 결정 된다. 

처음 출발할때는 TTL을 1로 하고 ICMP 메세지를 받고 부터는 TTL 값이 1씩 증가한다.

Traceroute를 실행하면 아래와 같이 hop의 주소와 다음 hop을 어디로 움직였는지가 나온다. 최종적으로는 마지막 도착지까지 나온다. (마지막에 표시된 ip를 브라우저 주소창에 입력하면 google.com로 접속이 가능하다.

![Untitled](/img/traceroute_ping_netstat/Untitled%203.png)

각 hop의 RTT(왕복시간)과 함께 전체 경로를 보여준다.(위 블로그에서는 ping이나 traceroute의 RTT는 보장되는 값은 아니라고 한다. 왜냐면 ping과 traceroute는 상대적으로 HTTP나 SSH보다는 우선순위가 낮기 때문이다.

traceroute 네트워크 이슈가 경로와 관련있다고 여겨질때 사용하면 적절한 툴이다.

# Netstat

Netstat는 특정 엔드포인트에서 연결된 네트워크를 다 보여주는 것으로 직접 본인의 컴퓨터에서 하게되면 열려있는 포트와 연결된 정보를 알 수 있다. 간혹 특정 서버가 켜져 있는데도 일부 포트로 연결시 refusion이 발생할 경우 그 특정 서버에서 netstat 실행하여 접속하려는 포트가 열려 있는지 안 열려있는지 확인이 가능하다.

![Untitled](/img/traceroute_ping_netstat/Untitled%204.png)

맥에서 netstat만 사용했더니 리스닝 하는 포트는 확인하기 어려웠다.

netstat의 옵션 커맨드를 추가하니 리스닝 포트만 추려서 볼 수 있었다.

더 자세한 옵션이 필요하면 이 링크로 들어가면 확인 할 수 있다. (맥 OS 기준)

[https://benohead.com/blog/2013/07/31/mac-netstat-list-ports-programs/](https://benohead.com/blog/2013/07/31/mac-netstat-list-ports-programs/)

![Untitled](/img/traceroute_ping_netstat/Untitled%205.png)

4번째 컬럼의 .다음 숫자들이 현재 로컬 서버에서 열려있는 포트 번호들이다. (*은 아무번호나 다 해당한다는 의미다. 쉽게 말해 0.0.0.0이라고 생각하면 된다.

기본적으로 네트워크 장애가 발생한다면 위의 세가지 상황에 맞게 사용하면 된다. 

클라우드를 운영하다보면 의외로 네트워크가 무척 중요하게 된다.

다음에는 Azure와 온프레미스 vpn연결에 있었던 시행착오를 한 번 업로드 해보겠다.