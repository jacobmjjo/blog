---
title: "Serverless"
date: 2019-10-06T00:27:48+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud"]
math: false
toc: false
---

### Serverless
**서버가 없다는 뜻이 아님**

단지 가상머신에 서버 설치후 서버를 관리할 필요가 없다는 뜻임  
서버 개수, 네트워크 종류, 서버 사양을 고려할 필요가 없음  

***
### Baas(Backend as as a Service)
서버확장, 보안성 등의 서버개발을 지원해줌.  

개발자들이 서버 개발 안해도 되고 서버의 이용자가 순식간에 늘어나도 알아서 확장 가능  
대표적으로 AWS Firebase가 있다.  

단점
1. 클라이언트 위주의 코드
2. 비용
3. 복잡한 쿼리 불가능함

***
### FaaS(Function as a Service)

FaaS는 프로젝트를 여러개의 함수로 쪼갬, 분산된 컴퓨팅 자원에 함수를 등록하고, 실행되는 횟수만큼 비용을 지불하는 방식  

특정한 이벤트 할 때만 실행.  

PaaS는 항상 실행 하지만 FaaS는 이벤트 때만 실행됨 그리하여 비용이 저렴함.  
대표적으로 AWS 람다가 있다.  
