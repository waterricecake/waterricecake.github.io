---
layout: post
title: "[쿠버네티스] Volume"
categories:
  - 학습
  - Kubernetes
tags:
  - Kubernetes
  - k8s
  - Volume
comments: true
---
# 1. 볼륨이란

## 1.1. 파드의 문제점

이미 띄운 파드에 문제가 생길 경우 쿠버네티스는 파드를 수정하지 않고 새로운 파드를 만들어 교체를 한다.

이럴 겨우 파드 교체시 기존 파드 내부의 데이터도 같이 삭제가 되게 되는데, 이때 MySQL 같은 DB의 데이터도 같이 삭제되게 된다.

이와 같은 문제를 해결하는 방법이 볼륨이다.

### 1.1.1. 예제

기타 매니페스트 파일은 생략하고 다음 디플로이먼트로 mysql을 띄웠을 때

```yaml
apiVersion: apps/v1  
kind: Deployment  
  
metadata:  
  name: mysql-deployment  
  
  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: mysql-db  
  
  template:  
    metadata:  
      labels:  
        app: mysql-db  
    spec:  
      containers:  
        - name: mysql-container  
          image: mysql  
          ports:  
            - containerPort: 3306  
          env:  
            - name: MYSQL_ROOT_PASSWORD  
              valueFrom:  
                secretKeyRef:  
                  name: mysql-secret  
                  key: mysql-root-password  
            - name: MYSQL_DATABASE  
              valueFrom:  
                configMapKeyRef:  
                  name: mysql-config  
                  key: mysql-database
```

## 1.2. 볼륨

볼륨이란 데이터를 영속적으로 저장하기 위한 방법이다.

쿠버네티스에서 2가지 종류로 나뉜다.

### 1.2.1. 로컬 볼륨

파드 내부의 공간 일부를 볼륨으로 활용하는 방식

이 방식은 파드가 삭제되면 데이터도 삭제되므로 거의 사용되지 않는다.

### 1.2.2. 퍼시스턴트 볼륨 (Persistent Volume, PV)

파드 외부의 공간 일부를 볼륨으로 활용하는 방식

파드가 삭제되는 것과 상관없이 데이터를 영구적으로 사용할 수 있다.

#### 1.2.2.1. 퍼시스턴트 볼륨 클레임 (Persistent Volume Claim, PVC)

파드는 직접적으로 PV에 연결할 수 없는데, 이 가운데 중개자 역할을 하는 것이 PVC이다.