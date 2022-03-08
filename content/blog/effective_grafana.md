---
title: "효율적인 그라파나 인프라 모니터링"
date: 2020-12-01T21:23:42+09:00
slug: ""
description: ""
keywords: ["Grafana"]
draft: true
tags: ["Grafana","Monitoring"]
math: false
toc: false
---

아래의 링크를 번역한 것이다.

[How to Do Effective Infrastructure Monitoring for Linux with Grafana](https://grafana.com/blog/2019/10/09/how-to-do-effective-infrastructure-monitoring-for-linux-with-grafana/)

각 매트릭 별로 쿼리를 알려준다

USE 모니터링에 필요한 

- 사용량
- 포화량

에 대한 쿼리가 담겨있다.

그리고 추가적으로 가장 하단에 에러 수위에 대한 쿼리도 첨부 하였다.

## CPU 사용량

![/img/effective_grafana/Untitled.png](/img/effective_grafana/Untitled.png)

## CPU 포화량

![/img/effective_grafana/Untitled%201.png](/img/effective_grafana/Untitled%201.png)

## 메모리 사용량

![/img/effective_grafana/Untitled%202.png](/img/effective_grafana/Untitled%202.png)

## 디스크 사용량

![/img/effective_grafana/Untitled%203.png](/img/effective_grafana/Untitled%203.png)

## 네트워크 사용량

![/img/effective_grafana/Untitled%204.png](/img/effective_grafana/Untitled%204.png)

## Error

```
http_requests_total 13.0
http_status_500_total 4.0

rate(http_status_500_total [1m]) / rate(http_requests_total [1m])
```