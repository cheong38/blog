---
draft: false
title: Minikube 설치
author: [Woojin]
date: 2020-08-27
image: ./minikube.jpg
imageAlt: Minikube logo
tags: ['minikube']
---

# Minikube

minikube 는 설치하면 쿠버네티스 클러스터를 PC 의 가상 머신에서 구동시킬 수 있다.

## 설치

[이 곳](https://kubernetes.io/ko/docs/tasks/tools/install-minikube/) 을 참고하여 운영체제에 맞게 설치하면 된다.

## 구동하기

```
$ minikube start
```

터미널에 위의 명령어를 입력하면 minikube 환경이 시작된다.

```
$ kubectl config get-contexts
```

그 후, 위의 명령어를 입력하면,

```
CURRENT   NAME         CLUSTER     AUTHINFO      NAMESPACE
...
*         minikube     minikube    minikube
...
```

위와 같이 minikube 클러스터가 생기고 현재 컨텍스트로 minikube 가 선택되어 있는 것을 확인할 수 있다.

몇 가지를 더 테스트해보자.

```
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   11m
kube-node-lease   Active   11m
kube-public       Active   11m
kube-system       Active   11m
```

```
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m
```

위와 같이 정상 구동중인 것을 확인할 수 있다.
