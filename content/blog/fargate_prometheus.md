---
title: "fargate에 프로메테우스 적용 테스트"
date: 2021-08-12T14:28:01+09:00
slug: ""
description: ""
keywords: ["Prometheus"]
draft: true
tags: ["Cloud","Prometheus","ECS"]
math: false
toc: false
---

결론은 못 쓰는 것 같다.

프로메테우스를 적용하기 전 컨테이너가 어떠한 환경인지 알아둘 필요가 있다. 

현 프로젝트에서는 ecs에서 fargate만 사용한다. 즉 ecs container를 사용하지않고 network만 사용한다. 

그렇기에 차이점 또한 존재한다. 

[AWS Container Orchestration 101: ECS vs Fargate vs EKS](https://www.dragonspears.com/blog/aws-container-orchestration-101-ecs-vs-fargate-vs-eks)

현 환경에서는 쿠버네티스와 프로메테우스를 연계하는 방법을 사용하기는 힘들다. 

만약 ecs container기반이라면 cadvisor를 각 클러스터에 설치하면 가능하다.

관련 자료는 아래 북마크를 보면 된다.

[Monitor AWS ECS with Prometheus and Cadvisor](https://medium.com/@lynnlin827/monitor-aws-ecs-with-prometheus-and-cadvisor-2d8b791a8b1c)

근데 fargate 네트워크만 사용하기에 해당 방법은 적용이 어렵다.

하지만 아래의 링크를 보니 뭔가 방법이 있는 듯한 자료를 발견 했다.

이건 나중에 시도해 보겠다.(해도 비용이 거의 서비스 만큼이나 나온다.)

[Prometheus service discovery for AWS ECS](https://tomgregory.com/prometheus-service-discovery-for-aws-ecs/#prometheus-resource)

그래서 AWS 공식 자료에서 제공하는 cloudcontainer와 프로메테우스 연계 방식으로 적용방식으로 관심을 가지게 되었다. 

아래가 그와 관련된 내용인데 관련 글은 [ASP.net](http://ASP.net) core (해당 링크른 [ASP.NET](http://ASP.NET) core 에 대한 자료다[https://jakeydocs.readthedocs.io/en/latest/conceptual-overview/aspnet.html](https://jakeydocs.readthedocs.io/en/latest/conceptual-overview/aspnet.html))에 프로메테우스 매트릭을 수집하는 방식이다.

[Amazon CloudWatch Prometheus metrics now generally available | Amazon Web Services](https://aws.amazon.com/ko/blogs/containers/amazon-cloudwatch-prometheus-metrics-ga/?sc_channel=EL&sc_campaign=Demo_Deep_Dive_2021_vid&sc_medium=YouTube&sc_content=Video8855&sc_detail=MANAGEMENT_GOVERNANCE&sc_country=US)

오리지널 프로메테우스랑은 다르게 수집할 수 있는 매트릭이 정해져있다. 

[(Optional) Set up sample containerized Amazon ECS workloads for Prometheus metric testing](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Sample-Workloads-ECS.html)

여기서부터 프로메테우스를 써야하는지에 대해 회의적인 생각이 들었다. 

우리가 알아야할 cpu 사용량이나 메모리 사용량등은 cloudcontainer로도 확인이 가능하다.

또한 saturation는 fargate에서 의미가 있는지 또한 각 클러스터당 한개의 task만 사용하고 있는 상황인지라 ec2와 비슷하게 모니터링을 해도 무방할 듯 싶다

[Performance Bottleneck : High CPU Utilization vs High CPU Saturation](https://performanceengineeringin.wordpress.com/2019/05/29/high-cpu-utilization-vs-high-cpu-saturation/)

[A Deep Dive into Kubernetes Metrics - Part 3 Container Resource Metrics](https://blog.freshtracks.io/a-deep-dive-into-kubernetes-metrics-part-3-container-resource-metrics-361c5ee46e66)

혹시나 해서 프로메테우스를 설치해봤지만 JMX나 nginx같은 소프웨어를 사용하는 클러스터한에서만 가능하기에 굳이 할 필요가 없는 것으로 생각된다.

cloud container에거 볼수 있는 매트릭 기준으로 golden signal을 만들면 1차적인 단계는 해결 될것으로 생각된다.