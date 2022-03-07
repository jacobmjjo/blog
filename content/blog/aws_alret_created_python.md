---
title: "파이썬을 이용하여 AWS 알람 생성"
date: 2021-03-16T00:06:47+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","AWS","Management"]
math: false
toc: false
---


AWS를 서비스를 이용하다보면 자주 사용하게 되는 서비스들이 있는데 그 중 하나가  CloudWatch이다.

CloudWatch는 AWS에서 제공해주는 모니터링 서비스로 EC2,RDS,ELB 등의 리소스 정보를 실시간으로 파악할 수 있게 해준다. 

무튼 이 CloudWatch(길어서 CW로 하겠다.)도 여러가지 부수적인 서비스가 있는데 여기서는 알람에 대해 다루겠다.

CW알람이란 말그대로 특정 모니터링 그래프에서 일정 임계치를 넘어가게 되면 자동으로 알람을 가게끔하는 서비스이다. 리소스 별로 또 해당 리소스의 지표 별로 설정이 가능하다. 알람 방식도 설정이 가능하다. 

알람 설정이 적을 경우 문제가 안되는데 알람설정이 많을 경우에는 생성작업이 좀 고달파진다.

이럴 경우에는 파이썬과 AWS CLI를 이용하면 금방 해결한다(코드 만드는 시간은 오래 걸렸다만....)

필요한 요소는 크게 4가지 이고 사용한 OS는 맥을 기준으로 하겠다.

- AWS CLI
- Jupyter Notebook(Anaconda)
- 파이썬 패키지들(pandas, **boto3**, openpyxl)
- AWS 액세스 키 및 비밀번호

먼저 알람을 어떤 식으로 만들지 생각해야 한다.

필자의 경우에는 알람 이름, 리소스 ID를  for loop을 이용해서 boto3에서 제공하는 알람생성 명령어내 설정에 정보를 자동기입 하려고 한다.

알람의 이름은 리소스 이름(태그로는 Name)에 지표와 임계치를 붙여서 생성하려고 하고 알람 생성에 필요한 CW 매트릭에는 추출한 리소스 ID를 기입하여 생성할 계획이다.

이 문서에서의 지표는 CPU 사용량이고 리소스는 EC2이다.

AWS는 CLI를 이용하여 리소스들의 정보를 가져오기 쉽게 끔 명령어를 제공해 준다.

가장 먼저 AWS CLI가 설치하고 설치가 되어있다면 알람을 만들려고 하는 계정으로 AWS CLI를 설정한다.

```bash
aws configure
```

위의 명령어를 입력하면 엑세스 키와 비밀번호 그리고 리전을 설정하라고 나온다.

내용에 맞게 입력한다

올바르게 액세스 키를 입력했다면 AWS CLI를 이용하여 인스턴스 이름, 인스턴스 ID를 가져와야 한다.

그럼 아래의 명령어를 이용하여 가져온 리소스 정보를 json 파일로 저장한다.  

```bash
aws ec2 describe-tags --filters "Name=resource-type,Values=instance" --output json > {맘에드는 이름}.json
```

해당 파일을 열어보면(열려면 터미널에서 cat으로 보거나 SublimeText로 열람해서 보면 된다. Sublime은 json 읽을 수 있는 패키지를 설치해야한다.) 한 인스턴스 ID임에도 태그 종류에 따라 여러개가 나오는 것을 확인 할 수 있다. 

필자는 인스턴스 ID가 중복일 필요가 없기 때문에 Pandas를 통해서 중복제거 할 것이다.

Pandas를 쓰는 이유는 데이터 관리 할 때 매우 편하기 때문이다. 필요한 컬럼만 뽑거나 값을 비교해서 새 컬럼을 만들거나, 시각화 등 매우 유용하기 때문이다. 

무튼 받은 json 파일은 딕셔너리의 딕셔너리의 딕셔너리 같은 형태가 많기에 정리가 필요하다. 

 

```python
import pandas as pd
import json
f = open('{방금 저장했더 describe_tags json 파일}.json', "r", encoding="utf-8")
array = json.load(f)
df = pd.json_normalize(array, record_path=['Tags'])
df.to_excel('{원하는 이름}.xlsx') #df를 엑셀파일로 저장한다. 
```

판다스를 import 한 뒤 해당 파이썬 코드를 사용하면 관계형 데이터처럼 변환이 가능하다.

여기서 중요한 점은 record_path 값이 왜 'Tags'인가이다. 

json 파일을 열면 딕셔너리의 딕셔너리 같은 형태인데 이것을 엑셀같은 관계형으로 바꿀려면 딕셔너리의 키들의 경로를 이해하고 record_path에다가 기입해야 한다. 다른 방법도 있겠지만 여기서는 이 방법으로 설명한다. 

만들고 나면 컬럼이 Key, instanceId, ResourceType, Value 이렇게 4가지 칼럼이 나오는 것을 볼 수 있다.

알람 생성에 필요한 내용은 instanceId 그리고 Value 중에 인스턴스 이름에 해당하는 값이다.

 Value 중에 인스턴스 이름에 해당하는 값이란 앞서 언급했던 인스턴스 ID 중복이란 연관되는데 Key 라는 행을 보면 Name과 기타 태그 값들을 볼 수 가 있을 것이다.이것을 의미하는 바는 태그의 키이고  Value는 동일한 열을 기준으로 태그의 키 값이다. 즉 동일한 열에서 Key가 Name일 경우에는 Value는 태그 Name에 해당하는 값이다. 

필요한 raw만 뽑을 수 있도록 pandas를 코딩해 보겠다. 

```python
tags = pd.read_excel('{describe_tags로 만들었던 엑셀파일}.xlsx') 
# 엑셀 저장이 불필요하면 바로 위에서 만든 'df'를 사용하면 된다.
newtag = tags.loc[(tags['ResourceType'] == 'instance') & (tags['Key'] == 'Name') ]
newtag
```

중복 instance가 사라지고 Key에는 Name만 남은 상태이다. 

알람 생성 매트릭에 필요한 정보는 준비가 다 되어있다. 

그럼 boto를 사용해서 알람 생성을 해보자.

샘플코드는 aws에서 공유해 준다. 샘플 코드 기반으로 수정하여 사용하면 된다.

코드에서 create_metric_alarm_CPU에 입력할 변수로 instance_name은 인스턴스 이름이고 instance_id는 인스턴스의 ID이다.  

[cloudwatch_basics.py](https://docs.aws.amazon.com/code-samples/latest/catalog/python-cloudwatch-cloudwatch_basics.py.html)

```python
class CloudWatchWrapper:
    """Encapsulates Amazon CloudWatch functions."""
    def __init__(self, cloudwatch_client):
        """
        :param cloudwatch_resource: A Boto3 CloudWatch resource.
        """
        self.cloudwatch_client = cloudwatch_client
    def create_metric_alarm_CPU(self,instance_name, instance_id):
        try:
            cloudwatch = self.cloudwatch_client
            alarm = cloudwatch.put_metric_alarm(
                AlarmName= instance_name +': CPU Utilization over 90 percent',
                ComparisonOperator='GreaterThanThreshold' #해당 문구는 임계점보다 더 클 경우이다.,
                EvaluationPeriods=1,
                MetricName='CPUUtilization', #CPU사용량
                Namespace='AWS/EC2',
                Period=#원하는 초단위 시간, 300은 5분임,
                Statistic='Average', #여기선 평균으로 함
                Threshold=# 임계점, 90.0으로 하면 90%를 의미함,
                ActionsEnabled=True, #이거를 false를 해놓으면 AlarmActions가 아무런 활동을 하지 않는다. 그러므로 꼭 True로 해야한다.
                AlarmActions=[#sns를 사용한다면 sns의 Arn주소를 입력하면 된다. 필자는 sns를 사용했다.],
                AlarmDescription='Alarm when server CPU exceeds 90%',
                Dimensions=[
                    {
                      'Name': 'InstanceId',
                      'Value': instance_id
                    }
                ]
            )  
            logger.info(
                    "Added alarm %s to track metric AWS/EC2,CPUUtilization.", instance_name)
            
        except ClientError:
            logger.exception(
                    "Couldn't add alarm %s to track metric AWS/EC2,CPUUtilization.", instance_name)
            raise
        else:
            return
```

해당 코드를 이용하면 쉽게 코드를 여러개 만들 수 있다.