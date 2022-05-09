---
title: "교차계정으로 타계정 S3에 VPC 엔드포인트로 접속 "
date: 2022-05-09T10:29:46+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: []
math: false
toc: false
---

클라우드 아키텍처를 설계하다 보면 타계정의 있는 서비스를 사용해야 하는 경우가 있다. 

AWS는 그러한 상황에 대비하여 ‘교차계정’ 이라는 서비스를 제공해 준다. 

교차 계정이란 쉽게 설명해서 타계정도 리소스를 사용할 수 있게 허락해주는 것이다.

허락만 하면 리소스를 자동으로 쓸 수 있냐... 그것도 아니다. 

계정 허가는 기본이고 사용하고자 하는 리소스에 맞춰 vpc를 연결해서 사용해야 한다. 

VPC를 연결하는 방식은 아래의 링크에 잘 나와있다. 

[3 options for cross-account VPC access in AWS](https://tomgregory.com/cross-account-vpc-access-in-aws/)

![Untitled](/img/croossAccount_s3_endpoint/Untitled.png)

이 글에서는 VPC연결 방법을 다룰건 아니고 VPC를 연결할 필요 없이 S3를 VPC endpoint 서비스를 이용해서 다이렉트로 프라이빗으로 연결하는 방법에 관한 내용이다.

먼저 VPC Endpoint를 이해할 필요가 있는데  VPC endpoint란 AWS에서 제공해주는 서비스 S3, DynanoDB, Kinesis stream 등을 인터넷이나 VPN없이 사설망으로 바로 접속할 수 있게 해주는 서비스이다.

(VPC Endpoint에 대해 궁금하면 아래 링크로 들어가면 된다.)

[VPC 엔드포인트](https://docs.aws.amazon.com/ko_kr/vpc/latest/privatelink/vpc-endpoints.html)

종류는 3가지가 있는데 이 글에서 다룰 내용은 S3연결에 적합한 Gateway VPC Endpoint이다. 

원하는 아키텍처는 타계정의 S3를 Endpoint 통해 내부망 접속하는 아키텍처이다.

그림으로 보면 아래와 같다.

![Untitled](/img/croossAccount_s3_endpoint/Untitled%201.png)

해당 아키텍처의 장점은 동일계정에서 엔드포인트를 쓰는 것과 큰 차이가 없는 통신 시간일 것이다. 

S3는 VPC와는 상관없이 생성하는 서비스이므로 동일계정에서 엔드포인트로 통해 접속하나 타계정의 엔드포인트로 접속을 하거나 큰 차이가 없을 것이다.(개인적인 생각)

하지만 타계정에서 접속하는 것이기에 타계정이 S3를 사용할 권한을 습득해야 한다.

교차계정 권한 주는 방식은 아래 링크에 자세히 나와있다. 필자는 가이드에서 첫번째 방법을 적용했다.

[Amazon S3 객체에 대한 교차 계정 액세스 제공](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cross-account-access-s3/)

먼저 S3권한에 타계정에서 사용할 수 있도록 JSON을 만들고 권한에 추가한다. 

(그 전에 ec2에 적용할 역할이 미리 준비 되어 있어야 한다. 필자는 s3 full로 했다.)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::{API를 사용할 타계정 번호}:role/{S3 사용 권한을 가진 ec2 역할}"
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::{s3 이름}/*"
        }
    ]
}
```

그리고 나서 타계정(S3 X)에 VPC Endpoint를 생성한다. 

![Untitled](/img/croossAccount_s3_endpoint/Untitled%202.png)

라우팅 테이블을 지정하는 것을 잊으면 안된다.

![Untitled](/img/croossAccount_s3_endpoint//Untitled%203.png)