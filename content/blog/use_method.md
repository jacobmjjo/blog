---
title: "USE Method"
date: 2020-12-01T21:53:24+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","Monitoring","Container","Management"]
math: false
toc: false
---

컨테이너 모니터링 구성 중 인프라 모니터링에 해당하는 방법이다.

원문은 아래 링크다.

[The USE Method](http://www.brendangregg.com/usemethod.html)

### 소개

서버를 사용하거나 컨테이너를 사용하든지 어떠한 구성의 경우에도 이슈를 빠르게 체크 할수 있게 해준다.

필요한 매트릭은 

- 리소스
- 사용량
- 포화량
- 에러 발생 수

여기서 이해하기 어려운 항목이 포화량이다. 사용량과는 차이가 있는데 

할당 받은 리소스를 넘어선 추가작업이다. 할당 받은 리소스는 큐로 표시한다.

위 USE 문서에서 추천하는 기본 적인 매트릭은 아래 표와 같다 

![Metrics](/img/use_method/Untitled1.png)

아래는 전략으로 권장 하는 진행 방식이다. 

![출처:[https://www.brendangregg.com/usemethod.html](https://www.brendangregg.com/usemethod.html)](/img/use_method/Untitled.png)

출처:[https://www.brendangregg.com/usemethod.html](https://www.brendangregg.com/usemethod.html)