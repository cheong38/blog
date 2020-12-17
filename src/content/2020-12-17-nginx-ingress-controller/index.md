---
draft: false
title: Minikube 로 Nginx Ingress Controller 사용하기 (with AWS ECR)
author: [Woojin]
date: 2020-12-17
image: ./nginx_ingress_controller.png
imageAlt: Nginx Ingress Controller Diagram
tags: ['kubernetes', 'nginx']
---

# 개요

쿠버네티스 클러스터 외부의 트래픽을 정해진 라우팅 규칙을 통해 내부 서비스로 연결해주는 
쿠버네티스 리소스를 [인그레스(ingress)](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/) 라고 한다.

인그레스가 작동하려면 인그레스 컨트롤러가 반드시 필요한데 본 포스팅에서는 Nginx Ingress Controller 를 사용하는 법을 다루려고 한다.

# 실습

## Minikube 설치하기

[지난 포스팅](https://cheong38.github.io/2020-08-27-minikube/index/) 을 참고해 설치힌다.

## Minikube 시작하기

```bash
$ minikube start --vm=true
```

`--vm=true` 옵션을 주지 않으면 ingress addon 을 설정하는 과정에서 에러가 발생한다.

만약 `--vm=true` 옵션으로 실행시 에러가 발생한다면, [VirtualBox](https://www.virtualbox.org/) 와 같은
가상머신이 설치되어있는지 확인해보자.

## ingress addon 설정하기

minikube 에서 ingress 를 사용하기 위해서는 addon 을 설정해 주어야 한다.

```bash
$ minikube addons enable ingress
```

## AWS ECR 의 이미지 사용할 수 있도록 설정

본 포스팅에서는 [AWS ECR](https://aws.amazon.com/ko/ecr/) 에 이미지를 올려놓은 경우를 가정하고 있다.

따라서 ECR 의 이미지를 받아올 수 있도록 Minikube 에 권한 등을 설정해야 한다.

이 역시 minikube addon 을 통해 설정할 수 있다.

자세한 설명은 [여기](https://minikube.sigs.k8s.io/docs/tutorials/configuring_creds_for_aws_ecr/) 를 참고한다.

```bash
$ minikube addons configure registry-creds
```

위 명령어 입력 시 아래와 같은 추가 입력을 받을 것이다.

```
Do you want to enable AWS Elastic Container Registry? [y/n]: y
-- Enter AWS Access Key ID: <put_access_key_here>
-- Enter AWS Secret Access Key: <put_secret_access_key_here>
-- (Optional) Enter AWS Session Token:
-- Enter AWS Region: <put_aws_ecr_region>
-- Enter 12 digit AWS Account ID (Comma separated list): <account_number>
-- (Optional) Enter ARN of AWS role to assume:

Do you want to enable Google Container Registry? [y/n]: n

Do you want to enable Docker Registry? [y/n]: n

Do you want to enable Azure Container Registry? [y/n]: n
✅  registry-creds was successfully configured

```

우리는 ERC 만 설정할 것이므로 `Do you want to enable AWS Elastic Container Registry? [y/n]` 에만 `y`를 선택하고,
나머지 `Do you want ~` 에는 `n` 을 선택한다.

`<>` 사이의 내용은 적절한 값으로 대체해 넣으면 된다. 빈 칸은 엔터를 치고 넘어가면 된다.

## Ingress 파일 배포

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: main-ingress
  namespace: main
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              serviceName: web
              servicePort: 80
          - path: /api
            pathType: Prefix
            backend:
              serviceName: api
              servicePort: 80
```

위의 예시와 같은 ingress 파일을 작성하여 배포해보자. 물론 위의 serviceName 과 servicePort 에 맞는 쿠버네티스 리소스가 설정되어 있어야 한다.

## 접속해보기

```bash
$ minikube ip
```

위의 명령어로 나온 IP 주소를 브라우저에 입력해 접속하면 배포한 서비스에 접근할 수 있을 것이다. 
