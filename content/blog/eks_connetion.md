---
title: "ECS 접속 방법"
date: 2022-02-08T13:28:08+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","AWS","Container"]
math: false
toc: false
---

호텔 모바일 키 체크인 아웃 서비스는 컨테이너를 사용한다.

AWS에서 제공하는 Fargate라는 서비스를 이용하는데 Fargate는 Networking, EC2 + linux, EC2 + Windows 이렇게 세가지를 선택할 수 있다.

모바일 서비스는 이 중 Networking을 사용하는데 다른 2개의 서비스와는 컨테이너를 구동하는 서버에 접근을 못한다.

보통 쿠버네티스는 클러스터가 있고 그 안에 노드 그룹이 구성되며 특정 노드 안에는 여러 pod가 작업한다. 여기서 pod가 컨테이너에 해당한다. 

networking 모드는 노드(ec2)를 거치지 않고 바로 pod로 진입하거나 명령어를 날릴 수 있다. 

pod에 명령어를 날릴려면 execute-command를 이용해야 한다.

진입한다는 가정하에 명령어를 쓴다면 아래 처럼 쓴다.

```bash
aws ecs execute-command --cluster {cluster name} --task {클러스터 ARN/task id} \
--container {cotainer name} --interactive --command "bin/bash"
```

위에서 “bin/bash” 가 클러스터 접속한다는 명령어다.

의외에도 arch나 기타 여러 명령어 사용이 가능하다.