---
draft: false
title: Docker CMD 와 ENTRYPOINT
author: [Woojin]
date: 2021-02-23
image: ./main.png
imageAlt: IP
tags: ['docker', 'cmd', 'entrypoint']
---

# Docker CMD vs ENTRYPOINT

docker 는 컨테이너를 사용해 어플리케이션을 구축하고, 배포할 수 있게 하는 소프트웨어이다.

본 포스팅에서는 docker 의 `CMD` 와 `ENTRYPOINT` 의 차이점에 대해 설명한다.

## Docker 기본 사용법

```bash
$ docker run ubuntu
```

위의 명령어를 사용하면 ubuntu 이미지를 다운로드하여 docker 컨테이너를 실행시킨다.

VM (Virtual Machine) 과 다르게 컨테이너는 OS 자체를 호스트하는 것은 아니다.
컨테이너는 웹서버나 데이터베이스의 인스턴스와 같은 특정 태스크나 프로세스를 실행하는 것을 의미한다.  

태스크가 끝나면, 컨테이너는 종료된다. 컨테이너는 컨테이너 내부의 프로세스가 살아있는 동안만 유지된다.

## CMD

```dockerfile
FROM alpine:%%ALPINE_VERSION%%

...

RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && addgroup -g 101 -S nginx \
    && adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx \
    && apkArch="$(cat /etc/apk/arch)" \
    && nginxPackages="%%PACKAGES%%
    " \
    && case "$apkArch" in \
        x86_64) \
# arches officially built by upstream
  ...            

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 80

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
```

[`docker-nginx` 의 github 에서 dockerfile](https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile-alpine.template) 을
살펴보면, 위와 같은 파일 내용을 확인할 수 있다.

마지막 줄에서 `CMD` 키워드를 확인할 수 있는데, `CMD` 는 컨테이너 첫 실행시 실행할 명령어를 의미한다.

### CMD 덮어쓰기

```bash
$ docker run ubuntu [COMMAND]
```

위의 명령어에서 [COMMAND] 부분을 실행하고자 하는 명령어로 대체함으로써 `CMD` 를 덮어쓸 수 있다.

```bash
$ docker run ubuntu sleep 5
```

위의 명령어는 ubuntu 이미지의 `CMD` 는 `bash` 인데 이를 `sleep 5` 즉, 5초 간 sleep 하는 명령어로 변경한 예시이다.

이를 Dockerfile 로 작성하려면 아래와 같이 작성할 수 있다.

```dockerfile
FROM ubuntu

CMD sleep 5
```

혹은

```dockerfile
FROM ubuntu

CMD ["sleep", "5"]
```

### CMD 작성법

`CMD` 는 두 가지 형태로 작성할 수 있다.

첫 번째는 `쉘폼` 이고 두 번째는 `JSON Array` 형식이다.

JSON Array 로 작성할 때 주의사항은 array 의 첫 번째 요소가 반드시 실행가능한 명령어여야 한다는 것이다.

`CMD ["sleep 5"]` 와 같은 식으로 작성해서는 안된다.

위의 dockerfile 을 저장하고

```bash
$ docker build -t ubuntu-sleeper .
$ docker run ubuntu-sleeper
```

를 실행하면, `docker run ubuntu sleep 5` 와 동일한 실행 결과를 확인할 수 있다.

그런데 여기서 하드코딩 되어 있는 5 를 바꾸고 싶으면 어떻게 해야 할까?

## ENTRYPOINT

우리는 위의 예제를 아래와 같이 사용하고 싶다

```bash
$ docker run ubuntu-sleeper 10
```

이렇게 사용할 수 있다면, 사용하려는 컨텍스트에 따라 수치 조정을 할 수 있게 되어 보다 유연하다고 할 수 있을 것이다.

이를 위해 `ENTRYPOINT` 인스트럭션을 도입하자.

```dockerfile
FROM ubuntu

ENTRYPOINT ["sleep"]
```

`ENTRYPOINT` 는 `CMD` 와 마찬가지로 컨테이너 실행 시 실행할 명령어이다.
다만, `CMD` 는 docker run 명령어의 파라미터로 아예 대체가 되지만,
`ENTRYPOINT` 는 파라미터가 뒤에 더해진다.

### ENTRYPOINT 기본값

```bash
$ docker run ubuntu-sleepr
```

만약 앞선 예제의 dockerfile 에서 파라미터를 지정하지 않고 실행하면 어떻게 될까?

```
sleep: missiong operand
```

위와 같은 에러메세지를 보게 된다.

이제 우리의 목표는 파라미터를 지정하지 않으면 초기의 예제처럼 5초를 sleep 하고 지정되면 지정된 시간만큼 sleep 하는 것이다.
이를 위해 우리는 `ENTRYPOINT` 와 `CMD` 인스터럭션을 조합하여 사용할 수 있다.

```dockerfile
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

위의 예제를 살펴보자.

```bash
$ docker run ubnutu-sleeper
```

파라미터를 지정하지 않고 사용하면 `CMD` 의 값이 `ENTYRPOINT` 값에 추가되어 `sleep 5` 가 실행된다.

```bash
$ docker run ubuntu-sleeper 10
```

파라미터를 지정하여 사용하면 `CMD` 가 덮어쓰여져 `sleep 10` 이 실행된다.
