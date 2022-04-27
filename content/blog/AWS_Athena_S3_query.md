---
title: "Athena에서 Cloud Trail과 S3 액세스 로그 보는 법 "
date: 2022-04-27T14:50:48+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["AWS", "S3"]
math: false
toc: false
---

특정 IAM의 액세스키가 S3를 계속해서 이용하고 있는 것이 확인 되었다. 이것 자체는 문제가 아닌데  
왜 사용하는지를 아무도 모른다는 것이 확인되어 문제가 되어버렸다.  
그래서 어디서 사용하는지 확인 해보려고 CloudTrail에서 조회를 해보니까 검색이 되질 않았다.  
혹시 몰라 정밀한 검색을 위해 Athena에서 쿼리를 실행해 보았다.  
하는 방법은 아래의 가이드 링크를 보고 참고했다.

[AWS CloudTrail 로그 쿼리](https://docs.aws.amazon.com/ko_kr/athena/latest/ug/cloudtrail-logs.html)  
[Analyze Security, Compliance, and Operational Activity Using AWS CloudTrail and Amazon Athena](https://aws.amazon.com/ko/blogs/big-data/aws-cloudtrail-and-amazon-athena-dive-deep-to-analyze-security-compliance-and-operational-activity/)  
[Athena 테이블을 사용하여 CloudTrail 로그 검색](https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-tables-search-cloudtrail-logs/)

찾으려고 하는 활동은 s3에서 한 활동이니 s3한에서만 찾도록 쿼리를 작성했다.  

```sql
SELECT *
FROM cloudtrail_logs
WHERE
    eventsource = 's3.amazonaws.com' AND
    userIdentity.accessKeyId like 'ACCESSKEYID_HERE'
```

다만 여태껏 저장된 모든 Cloudtrail 기록을 다보면 비용이 크게 든다 그렇기에 파티셔닝해서 Athena 테이블을 만들어야 한다.  
파티셔닝 하는 방법은 아래 링크를 보면 된다.

[AWS CloudTrail 로그 쿼리](https://docs.aws.amazon.com/ko_kr/athena/latest/ug/cloudtrail-logs.html#create-cloudtrail-table-partition-projection)

4월 23일부터 4월 25일 3일간의 기록만 필요하므로 기간에 맞춰서 파티셔닝하고 테이블을 생성했다.  
그리고 쿼리를 실행해 보았는데 아무것도 나오지 않았다.....

![Untitled](/img/aws_athena_s3_query/Untitled.png)

IAM 콘솔에서는 사용했다라고 나와있고 Cloudtrail에는 조회가 안되는 난감한 상황이다.  

결국 찾아보니 Cloudtrail은 버킷단에서 발생한 이벤트 예를 들어 버킷 생성, 버킷 삭제 등은 조회가 되지만 더 세세한 영역인 object 영역에서의 활동은 Cloudtrail에는 저장되지 않는다고 한다.  
대신 Object 활동은 S3 액세스 로깅을 활성화 하면 볼 수 있다고 하니 해당 방법을 적용해 보기로 했다.  

[Athena를 사용하여 Amazon S3 서버 액세스 로그 분석하기](https://aws.amazon.com/ko/premiumsupport/knowledge-center/analyze-logs-athena/)  
[Amazon S3 서버 액세스 로깅 사용 설정](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/enable-server-access-logging.html)

어떤 S3에서 액세스키를 이용하는지 확인이 어려우니 의심되는 S3부터 액세스 로깅을 활성화 했다.  
그리고 Athena에 테이블을 만들었다.  

여기서 SELECT 쿼리를 만드는 과정 중 한 가지 어려운 점이 있었다.  
컬럼중에 액세스키를 가지고있는 컬럼이 없었다.  Remote IP 주소나 요청자에 대해서는 있어도 액세스키는 별도로 없다.  
그래서 할 수 있는 방법은 IAM에 표시된 시간 기준으로 범위를 정해 모든 활동을 보는 방법이었다.  
쿼리문 작성시 parse_datetime문을 이용해서 기간을 설정한뒤 쿼리를 실행했다.

![Untitled](/img/aws_athena_s3_query/Untitled%201.png)

결과는 성공적이었다.
  
원인은 특정 인스턴스가 aws cli로 S3에 업로드 하는 과정에서 문제의 액세스 키를 사용하는 것이었다.  
S3 액세스 서버 로깅으로 다행히 사용 이유를 알 수 있었고 대안으로 업로드를 시도하는 인스턴스에 S3 권한을 부여하므로 문제를 해결 할 수 있었다.