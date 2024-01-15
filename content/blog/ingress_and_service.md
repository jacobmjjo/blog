---
title: "인그레스와 서비스"
date: 2024-01-15T09:00:10+09:00
slug: ""
description: ""
keywords: ["ingress", "service"]
draft: true
tags: ["AWS","Network","k8s","ingress","service"]
math: false
toc: false
---

# 인그레스와 서비스

날짜: May 25, 2023

EKS에 인그레스를 적용하려고 찾아보니 몇가지 알게된 기술적인 내용이 있어 정리해서 글 써보려고 한다.

시작하기 앞서 우선 인그레스에 대한 정의는 공식사이트에서는 아래처럼 설명한다. 

> **인그레스란?**
[인그레스](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#ingress-v1-networking-k8s-io)는 클러스터 외부에서 클러스터 내부 [서비스](https://kubernetes.io/ko/docs/concepts/services-networking/service/)로 HTTP와 HTTPS 경로를 노출한다. 트래픽 라우팅은 인그레스 리소스에 정의된 규칙에 의해 컨트롤된다.
[https://kubernetes.io/ko/docs/concepts/services-networking/ingress/](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/)
> 

즉 외부에서 클러스터로 HTTP와 HTTPS로 접근할때 직접 Pod로 접속하는 것이 아닌 중간에서 접속 경로 노출과 라우팅을 해주는 역할이다.

사실 이런 비슷한 역할을 해주는 기능이 이미 있다. 

Kubernetes의 서비스라는 기능으로 서비스의 주요 기능은 직접적으로 Pod의 주소로 접근하지 않아도 어플리케이션에 접속할 수 있게 해준다.

이 서비스는 여러가지 타입으로 만들 수 있는데 ClusterIP, NodePort, 로드밸런서이다. 

로드밸런서로 만들시 ClusterIP와 NodePort도 자동적으로 같이 만들어진다.

AWS에서 K8s를 서비스로 만들시 콘솔 화면으로 봤을때는 인그레스와 서비스가 비슷하게 보인다.

 

다만 현실적으로 인그레스를 쓰는 경우가 많은데 이유는 많은 어플리케이션들을 핸들링하기가 힘들기 때문이다.

특히나 API 엔드 포인트가 여러개인 경우는 각 엔드포인에 맞게 일일히 로드밸런서를 만들어야 하는데 이게 여간 쉬운일은 아니다.

이 문제를 해결할 수 있는 방법이 인그레스이다.

아래의 코드를 보면 path 또는 주소별로 다른 서비스에 라우팅 할수 있도록 설정된걸 확인 할수 있다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: some-ingress
    annotations:     
      alb.ingress.kubernetes.io/load-balancer-name: ingress
      alb.ingress.kubernetes.io/target-type: ip
      alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  ingressClassName: alb
  rules:
    - host: my.example.com
      http:
        paths:
          - path: /some-path
            pathType: Prefix
            backend:
              service:
                name: service-a
                port:
                  number: 80
          - path: /some-other-path
            pathType: Exact
            backend:
              service:
                name: service-b
                port:
                  number: 81
    - host: '*.example.com'
      http:
        paths:
          - path: /some-path
            pathType: Prefix
            backend:
              service:
                name: service-c
                port:
                  number: 82 
    - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-d
            port:
              number: 80

```

 위 코드에서는 호스트에 따라 “my.example.com”과 “*.example.com”그리고 타 모든 호스트에 따라 다른 서비스로 라우팅하는 것을 확인 할 수 있다.

또한 path 부분에서 “/some-path”와”/some-other-path”에 따라 다른 서비스로 라우팅하는 것 또한 확인 가능하다.

여기서 중요한 게 AWS에서는 ingress를 두가지로 사용할 수 있고 이 2가지의 차이점 때문에 이 글을 쓰게 되었다.

첫번째는 AWS Loadbalancer Controller를 이용한 방식이고 다른 하나는 Nginx Ingress Controller를 이용한 방식이다.

이 두가지 방식의 아키텍처 부분에서의 차이점은 라우팅을 외부에서 처리하냐와 내부에서 처리하냐이다. 

외부 처리는 “외부 로드밸런서”가 처리해주고 내부는 “내부 리버스 프록시 파드”가 처리 해준다.

AWS 공식 블로그에서 제공해주는 아키텍처는 아래와 같다.

![외부 로드밸런서처리 
[https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/](https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/)](/img/ingress_and_service/Untitled.png)

외부 로드밸런서처리 
[https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/](https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/)

![내부 리버스 프록시 처리
[https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/](https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/)](/img/ingress_and_service/Untitled%201.png)

내부 리버스 프록시 처리
[https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/](https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/)

2019년 공식 블로그에는 NLB + Nginx Ingress Controller가 더 적합할 때가 있다고 한다.

- 첫번째는 모든 네임 스페이스의 모든 인그레스 이벤트를 수신해서 모든 수신 규칙, 호스트 및 경로를 포함하는중앙 집중식 라우팅 파일을 사용할 수 있다.
- 두번째 이유는 여러 환경에 대한 여러 인그레스 객체 또는 동일한 네트워크 부하 분산기를 사용하는 네임스페이스도 가질 수 있다.
- 세번째는 NLB 자체에 대한 장점으로
    - 고정 IP 사용
    - 더 많은 확장성
    - 영역 고립
    - 출발지 주소 보존
    - 긴 시간 유지 가능한 TCP 연결
    - ALB대비 대역 사용량 감소

자세한 내용은 아래의 링크에서 확인이 가능하다.

[Using a Network Load Balancer with the NGINX Ingress Controller on Amazon EKS | Amazon Web Services](https://aws.amazon.com/ko/blogs/opensource/network-load-balancer-nginx-ingress-controller-eks/)

또한 아래의 링크는 두가지 타입의 인그레스를 설정하는 방식을 간략하게 가이드 해주는 내용이다.

[Provide external access to Kubernetes services in Amazon EKS](https://repost.aws/knowledge-center/eks-access-kubernetes-services)

 

ingress controller 관련한 AWS 공식 블로그이다.

이 글은 아래의 링크 자료를 대부분 참고했다.

[Exposing Kubernetes Applications, Part 1: Service and Ingress Resources | Amazon Web Services](https://aws.amazon.com/ko/blogs/containers/exposing-kubernetes-applications-part-1-service-and-ingress-resources/)

