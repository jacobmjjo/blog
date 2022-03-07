---
title: "RDS freeable memory 가 지속적으로 감소하는 이유"
date: 2021-06-07T00:06:57+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: []
math: false
toc: false
---


캡쳐해주신 이미지는 DB 메모리 여유량 입니다

MariaDB는 데이터 작업을 하기 위해 버퍼와 캐시를 할당 합니다.

위 작업에서 사용 가능한 메모리의 80%~90%를 사용합니다..

세션작업을 하면 할수록 테이블을 만들어야 하므로 메모리 여유량이 감소합니다.

비록지속적으로 감소 할지라도 세션이 종료되면 **자동으로 메모리를 확보하는 기능**이 있어 운영 상으로는 걱정 안 하셔도 됩니다.

실제로 모니터링을 보시면 어떠한 시점에서 메모리를 확보하는 것을 파악 하실 수 있습니다.

[Amazon RDS for MySQL 또는 MariaDB의 여유 메모리 수준이 낮은 문제 해결](https://aws.amazon.com/ko/premiumsupport/knowledge-center/low-freeable-memory-rds-mysql-mariadb/)

[MySQL :: MySQL 5.7 Reference Manual :: 8.12.4.1 How MySQL Uses Memory](https://dev.mysql.com/doc/refman/5.7/en/memory-use.html)

[What is "freeable memory"?](https://serverfault.com/questions/208445/what-is-freeable-memory?rq=1)

[Best practices for Amazon RDS](https://docs.amazonaws.cn/en_us/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html#CHAP_BestPractices.MySQLStorage)