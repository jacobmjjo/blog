---
title: "네트워크 지연 종류"
date: 2019-12-16T23:24:02+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Network"]
math: false
toc: false
---


지연에는 크게 3가지 종류가 있다.
1. 노드 처리 지연
2. 큐잉 지연 
3. 전파 지연
이다.


__노드 처리 지연(nodal processing delay)__
- 라우터에서의 처리 지연
- 패킷의 헤더를 조사하고 어느 출력 링크로 보낼지 결정하는 시간에 따른 지연

***
**__큐잉 지연(queuing delay)__**

- 패킷이 큐에서 출력링크로 전송되는 과정에서 생기는 딜레이  

> 여기서 큐란 패킷이 라우터에서 빠져나가지 못할 경우 라우터에 패킷을 저장하는 **방법**을 뿐이다. 큐는 패킷을 저장 하지않고 그저 FIFO(FIFO))로 나열만 한다. 즉 큐에는 패킷이 없고 패킷 주소만 가지고 있다.

***
__전송 지연( transmission delay)__
    
- 비트를 링크로 내보내는데 생기는 지연
- La/R(length / bandwidth)
- *트랙픽 강도 와 La/R 크기에 따른 상관관계*
    - R= 링크 전송률 비트가 큐에서 나가는 비율(bits/sec)
    - La = 비트가 큐에 도착하는 평균율
    - L = 패킷 크기(bits)
    - a = 패킷이 큐에 도착하는 평균율(packets/sec)
- La/R > 1 = 큐잉 지연 무한대
- La/R ~ 1 = 큐잉 지연 무지 커짐
- La/R < 1 = 큐잉 지연 적음

***
__전파 지연(propagation delay)__
- 물리적인 이유(와이어 재료, 와이어 손상 등)로 생긴 딜레이
- 링크에서 다음 라우터까지 전파하는데 발생하는 지연
- 링크 길이와 상관없음 

***
__End to End Delay__

d(end-end) = N (d(proc) + d(trans) + d(prop))
