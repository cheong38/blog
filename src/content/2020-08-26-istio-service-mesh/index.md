---
draft: false
title: Istio 서비스 매쉬 (service mesh)
author: [Woojin]
date: 2020-08-26
image: ./istio-logo.jpg
imageAlt: The istio logo
tags: ['istio']
---

# `Istio`
많은 조직에서 마이크로서비스를 채택함에 따라 한 조직 내에도 다양한 배포 환경이 존재하게 되었다.
[`istio`](https://istio.io/latest/) 는 이러한 배포의 복잡도를 줄이는 것을 도와준다.
`istio` 는 오픈 소스 서비스 매쉬(service mesh)이다.

## 서비스 매쉬 (Service Mesh)
서비스 매쉬(Service Mesh)는 서비스 간 통신의 규모가 커짐에 따라 서비스 간 통신을 추상화하여 안전하고 신뢰할 수 있게 만드는 인프라 계층의 아키텍처이다.
서비스 메쉬(Service Mesh)에서는 서비스의 호출이 서비스에 딸린 프록시(proxy)들끼리 이루어지게 된다.
`istio` 는 서비스 메쉬 전체에 대한 행위적 인사이트와 운영적인 제어를 제공한다.

### Istio 서비스 메쉬 (Service Mesh)
`istio` 를 사용하게 되면 각 서비스는 [envoy proxy](https://www.envoyproxy.io/) 를 [사이드카 패턴](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/sidecar)으로 각 pod 에 주입하게 된다.
`istio` 는 로드 밸런싱, 서비스 간 인증, 모니터링과 같은 기능을 쉽게 가능하도록 만들어준다.
`istio` 는 마이크로서비스 간 통신을 모두 가로채 `istio` 제어 플레인(plane)을 통해 설정과 제어가 가능토록 해준다.
다음과 같은 기능을 포함한다.

- HTTP, gRPC, 웹소켓, TCP 트래픽에 대한 자동 로드 밸런싱
- 라우팅 규칙, 재시도, failover 와 fault injection 과 같은 트래픽 행위의 상세한 설정
- 정책 및 API 설정을 pluggable 하게 제공(접근 제어, 속도 및 할당량 제한 등을 지원)
- 메트릭, 로그, 트래픽 추적 등을 자동으로 지원
- identity-based 인증으로 클러스터 내 서비스 간 통신을 안전하게 지원

![Istio Architecture](./istio-mesh.svg)

### Istio 설치

먼저 실행 환경을 위해 [minikube 를 설치](/minikube)하자

[이 곳](https://istio.io/latest/docs/setup/getting-started/)을 참고하여 `istio` 를 다운로드하고 `istioctl` 명령어를 등록하자.

그 후 아래 명령어를 실행하자.

```
$ istioctl install --set profile=demo
Detected that your cluster does not support third party JWT authentication. Falling back to less secure first party JWT. See https://istio.io/docs/ops/best-practices/security/#configure-third-party-service-account-tokens for details.
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete
```

설치가 완료되었다는 메세지를 받았다.
클러스터에 어떤 변화가 생겼는지 살펴보자.

```
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   20m
istio-system      Active   50s
kube-node-lease   Active   20m
kube-public       Active   20m
kube-system       Active   20m
```

`istio-system` 이라는 새로운 namespace 가 생긴 것을 확인할 수 있다.
이 namespace 에는 어떤 서비스들이 있는지 확인해보자.

```
$ kubectl get services -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
istio-egressgateway    ClusterIP      10.105.158.180   <none>        80/TCP,443/TCP,15443/TCP                                                     46s
istio-ingressgateway   LoadBalancer   10.111.101.97    <pending>     15021:31597/TCP,80:30532/TCP,443:30828/TCP,31400:30090/TCP,15443:31940/TCP   46s
istiod                 ClusterIP      10.105.52.77     <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP,853/TCP                                60s
```

egress gateway, ingress gateway, istiod 등의 서비스가 있는 것을 확인할 수 있다.
이들에 대해서는 다음에 다시 자세히 알아보자.
pod 들도 생성되었는지 확인해보자.

```
$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-695f5944d8-lcwhw    1/1     Running   0          6m41s
istio-ingressgateway-5c697d4cd7-zvb74   1/1     Running   0          6m41s
istiod-59747cbfdd-x7bgg                 1/1     Running   0          6m55s
```

서비스 별로 하나의 pod 들이 존재하는 것을 확인할 수 있다.

`istio` 가 정상적으로 설치된 것을 확인할 수 있었으면 다음 포스팅에는 실질적인 `istio` 의 사용을 알아보고자 한다.
