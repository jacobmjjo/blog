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

이 글에서는 VPC연결 방법을 다룰건 아니고 VPC를 연결할 필요 없이 S3를 VPC endpoint 서비스를 이용해서 다이렉트로 프라이빗으로 연결하는 방법에 관해 다룰거다.  

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

실습에 앞서 S3가 없는 타계정을 A계정이라고 하고 S3를 소유한 계정을 B계정이라고 지정하고 실습을 진행해보겠다.  
먼저 B계정에 있는 S3내 권한에 A계정에서 사용할 수 있도록 정책을 JSON으로 만들고 권한에 추가한다.  
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

그리고 나서 A계정에 VPC 엔드포인트를 생성한다. 
![Untitled](/img/croossAccount_s3_endpoint/Untitled%202.png)

라우팅 테이블을 지정하는 것을 잊으면 안된다.
![Untitled](/img/croossAccount_s3_endpoint/Untitled%203.png)

필자는 여기서 JMJ-PRI-RTB(private subnet)만 라우팅 테이블에 추가했다.  
엔드포인트 목록에서 아래와 같이 엔드포인트 정보가 표시되면 정상적으로 만들어 진 것이다.  
![Untitled](/img/croossAccount_s3_endpoint/Untitled%204.png)

그리고 나서 A계정의 EC2로 접속하여 S3에 있는 자료를 다운 받아 보겠다.  
![Untitled](/img/croossAccount_s3_endpoint/Untitled%205.png)

정상적으로 다운 받은 것을 확인 할 수 있다. 

헌데 현 환경에서는 VPC 엔드포인트를 지워도 다운 받는 것에는 문제가 없다.  
만약 지우게 된다면 인터넷 망(퍼블릭)으로 받을 수 있기 때문이다.  
그렇다면 VPC 엔드포인트가 있다면 프라이빗 망을 통한다는 사실은 어떻게 확인 할 수 있을까?  

2가지 방식으로 확인이 가능하다.  
1. S3 권한에 특정 VPC 엔드포인트만 허가
2. S3 액세스 서버 로깅을 활성화하여 remote ip  확인

첫번째 방식은 간단하다.  
실습 초반에 추가했던 JSON 정책에 아래의 코드를 추가하면 된다.

```json
{
			"Sid": "Access-to-specific-VPCE-only",
			"Effect": "Deny",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": [
				"arn:aws:s3:::{S3 이름}",
				"arn:aws:s3:::{S3 이름}/*"
			],
			"Condition": {
				"StringNotEquals": {
					"aws:SourceVpce": "vpce-{엔드포인트 id}"
				}
			}
		}
```

위 코드에서 엔드포인트 id를 별도의 다른 id로 입력을 하고 EC2에서 전과 같이 다운로드를 시도하면  
![Untitled](/img/croossAccount_s3_endpoint/Untitled%206.png)

403 에러가 발생한다.  
즉 등록한 VPC endpoint를 제외한 모든 VPC 엔드포인트는 거부하는 효과가 생긴다.

2번째 방법은 S3 액세스 서버에서 확인이 가능한데 
관련된 블로그 글은 아래 링크에서 확인이 가능하다.

[Athena에서 Cloud Trail과 S3 액세스 로그 보는 법](https://josephmjjo.github.io/blog/aws_athena_s3_query/)

VPC 엔드포인트를 지우면 Remote Ip가 public ip로 표시되고 VPC 엔드포인트를 설치하면 private ip로 표시가 된다.  
아래의 그림이 Athena에서 S3 엑세스 로그를 조회한 결과다  
![Untitled](/img/croossAccount_s3_endpoint/Untitled%207.png)

remote ip 부분을 보면 public ip와 private ip가 번갈아 보이는 것을 확인 할 수 있다.  
VPC 엔드포인트를 지웠을 때는 52로 ip가 시작하고 VPC 엔드포인트가 있을때는 10으로 ip가 시작한다.  
VPC 엔드포인트가 있다면 자동적으로 내부망을 사용한다는 것을 확인 할 수 있다.  
이로써 VPC 엔드포인트로 타계정 S3에 접속이 가능한 것을 알 수 있다.
