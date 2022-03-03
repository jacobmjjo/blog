---
title: "쿠버 삽질"
date: 2020-09-02T22:06:47+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: []
math: false
toc: false
---
# 20200902 쿠버 삽질

첨 클러스터를 만들시 프록시 문제가 발생한다. 원인을 파악하지는 못했고 우선 하나를 만든 후 노드를 추가해야 한다.

대쉬보드를 추가하면 브라우저에서 화면을 클러스터의 상태를 직관적으로 볼 수 있다.

```jsx
minikube start or minikube start --nodes 2 -p haha
minikube node add
minikube dashboard or minikube dashboard haha
```

여러 상태를 확인한다

```bash
kubectl get nodes
kubectl cluster-info
```

해당 명령어로 helm 설치 여부 파악

```bash
helm list

```

```yaml
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo ls
```

helm으로 prometheus 설치 

```bash
helm install stable/prometheus
```

그리고 설정을 바꿔야 하는데 보통 

프로메테우스 서버가 먹통일것이다.

![/img/prometheus_error.png](/img/waste_of_time_mini_kube/prometheus_error.png)

이 경우에는 디플로이먼트로 확인하고  아래 이미지의 영역의 fsGroup 와 runAsGroup 그리고 runUser 값을 변경한다.

```bash
kubectl get deployment 
kubectl edit deploy [프로메테우스 서버 디플로이먼트 이름]
```

![/img/kubectl_edit_deploy.png](/img/waste_of_time_mini_kube/kubectl_edit_deploy.png)

또 다른 방법으로는 네임 스페이스를 monitoring으로 하고 다운 받을 프로그램은 stable/prometheus로 한다.

(이 부분은 확인이 더 필요하다)

```bash
helm install metrics -n monitoring stable/prometheus-operator
```

제대로 설치 되었는지 네임스페이스 기준으로 확인

```bash
kubectl get all -n monitoring
```

여기서 가장 중요한 yaml파일 수정 명령어가 나온다.

```bash
kubectl edit service/metrics-grafana -n monitoring
```

최 하단에 type 키를 보면 원래는 Cluster IP로 되어있는데  NodePort로 변경해야 접속이 가능하다.

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: metrics
    meta.helm.sh/release-namespace: monitoring
  creationTimestamp: "2020-09-02T07:29:40Z"
  labels:
    app: prometheus-operator-prometheus
    app.kubernetes.io/managed-by: Helm
    chart: prometheus-operator-9.3.1
    heritage: Helm
    release: metrics
    self-monitor: "true"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:meta.helm.sh/release-name: {}
          f:meta.helm.sh/release-namespace: {}
        f:labels:
          .: {}
          f:app: {}
          f:app.kubernetes.io/managed-by: {}
          f:chart: {}
          f:heritage: {}
          f:release: {}
          f:self-monitor: {}
      f:spec:
        f:ports:
          .: {}
          k:{"port":9090,"protocol":"TCP"}:
            .: {}
            f:name: {}
            f:port: {}
            f:protocol: {}
            f:targetPort: {}
        f:selector:
          .: {}
          f:app: {}
          f:prometheus: {}
        f:sessionAffinity: {}
    manager: Go-http-client
    operation: Update
    time: "2020-09-02T07:29:40Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      
...
        
    manager: Go-http-client
    operation: Update
    time: "2020-09-02T07:29:40Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:spec:
        f:externalTrafficPolicy: {}
        f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-09-02T08:34:43Z"
  name: metrics-prometheus-operato-prometheus
  namespace: monitoring
  resourceVersion: "4565"
  selfLink: /api/v1/namespaces/monitoring/services/metrics-prometheus-operato-prometheus
  uid: 4101173a-7c7b-4a9c-addb-7db6aa83ff34
spec:
  clusterIP: 10.96.205.170
  externalTrafficPolicy: Cluster
  ports:
  - name: web
    nodePort: 31354
    port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus
    prometheus: metrics-prometheus-operato-prometheus
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

근데 중요한건 프로메테우스가 노드 ip쓰는지 봐야 한다 이걸 아직 못했다.

로컬에서 config 파일을 봐야 한다. 아래 경로로 들어가서 노드의 ip를 확인해야한다.

```bash
cd $HOME/.minikube/machines
```

그리고 브라우저에서 minikube ip와 nodeport(여기서는 31354) 입력하면 그라파나로 접속이 가능하다.

그라파나도 동일하게 작업이 가능하다.

아이디는 admin이고 비밀번호는 찾아야한다...(여기서는 prom-operator)

위와 똑같은 것을

```bash
kubectl edit service/metrics-grafana -n monitoring
```

그라파나 비밀번호 알아내기

```bash
kubectl get secret --namespace monitoring metrics-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```