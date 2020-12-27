---
draft: false
title: Kubernetes Nginx Ingress Controller 선택적으로 Auth(인증) 하기
author: [Woojin]
date: 2020-12-28
image: ./nginx-auth.png
imageAlt: Nginx Icon with Lock
tags: ['kubernetes', 'nginx', 'nginx ingress controller', 'auth', 'microservice']
---

# 개요

[지난 포스팅](/2020-12-19-nginx-ingress-controller-auth/index/) 에서 kubernetes 에서
nginx ingress controller 를 활용하여 auth 를 진행하는 법을 살펴보았다.

실제 서비스를 개발하다보면, 모든 경로에 대해서 인증을 진행해야 하는 것이 아니라 일부 경로는 인증을 해야 하고, 일부 경로는 인증을 하지 않아야 하는 경우가 생긴다.

본 포스팅에서는 nginx ingress controller 에서 선택적으로 인증을 하는 방법에 대해 살펴본다.

# 구현하기

```yaml
# auth-ingress.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  # Some metadata
  annotations:
    nginx.ingress.kubernetes.io/auth-url: http://auth.default.svc.cluster.local/auth/authz
    nginx.ingress.kubernetes.io/auth-method: POST
    nginx.ingress.kubernetes.io/auth-response-headers: x-auth-role,x-auth-id
spec:
  rules:
    - http:
        paths:
          - path: /main
            pathType: prefix
            backend:
              serviceName: main
              servicePort: 80
    # other services specs
```

위의 ingress 에 정의된 spec 의 경로들은 모두 해당하는 서비스에 요청하기 전에 `http://auth.default.svc.cluster.local/auth/authz`
에 요청을 보내 인증을 진행하고, 인증이 올바르게 된 경우에는 타겟 서비스에 요청이 전달되게 된다.
자세한 내용은 [지난 포스팅](/2020-12-19-nginx-ingress-controller-auth/index/)을 참고하자.

이 때, `main` 이라는 서비스가 존재하고 이 중, 다른 경로에 대해서는 인증을 진행해야 하지만,
`/main/products` 경로에 대해 인증을 하지 않아야 하는 경우를 가정해보자.

`auth-snippet` 이라는 어노테이션을 통해 위의 문제를 해결할 수 있다.

```yaml
# auth-ingress.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  # Some metadata
  annotations:
    nginx.ingress.kubernetes.io/auth-url: http://auth.default.svc.cluster.local/auth/authz
    nginx.ingress.kubernetes.io/auth-method: POST
    nginx.ingress.kubernetes.io/auth-response-headers: x-auth-role,x-auth-id
    nginx.ingress.kubernetes.io/auth-snippet: |
      if ( $request_uri ~ "/main/products" ) {
        return 200;
      }
spec:
  rules:
    - http:
        paths:
          - path: /main
            pathType: prefix
            backend:
              serviceName: main
              servicePort: 80
    # other services specs
```

`nginx.ingress.kubernetes.io/auth-snippet` 가 선언된 부분을 보자.

요청 URI 가 `/main/products` 이면 바로 200 코드를 리턴하고 있다.

이로써, `/main/products` 경로로 요청이 들어오면,
`http://auth.default.svc.cluster.local/auth/authz` 로 인증을 요청하는 것이 아니라 바로 200 상태코드를 리턴하게 되고,
인증이 성공된 것으로 간주되어 타겟 서비스로 요청이 전달되게 된다.

```
if ( $request_uri ~ "/main/products" ) {
  return 200;
}
```

이 부분은 nginx 문법을 사용한다. 특정 path 로 시작하는 것, 끝나는 것, 그 외 nginx 설정에서 지원하는 방법을 지원하니,
다양한 방법으로 다양한 경로에 대한 설정을 할 수 있을 것이다.
([이 아티클](https://www.programmersought.com/article/99462844690/)에도 잘 나와있다.)

다만, 추후 유지보수를 위해 주석을 필히 같이 남겨주는 것이 좋을 것 같다.

# 마무리

[nginx ingress controller 에 대한 사용법](/2020-12-17-nginx-ingress-controller/index/)
그리고 [nginx ingress controller 를 통해 auth 서비스를 설정하는 법](/2020-12-19-nginx-ingress-controller-auth/index/),
오늘은 선택적으로 인증을 진행하는 법에 대해 다루었다.

이 정도면, 일반적인 케이스의 서비스를 개발하는데는 무리가 없을 것이라 생각된다.
