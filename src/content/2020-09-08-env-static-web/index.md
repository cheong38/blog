---
draft: false
title: 정적 웹페이지(Static web)에 환경 변수 사용하기
author: [Woojin]
date: 2020-09-08
image: ./nginx.png
imageAlt: Nginx logo
tags: ['nginx']
---

# 개요

앱을 작성하고 빌드하다보면 환경 별로 서로 다른 값을 사용해야 하는 값들이 있다.
이러한 설정 값을 환경 변수라고 부른다.
일반적으로 development, staging(qa), production 과 같이 환경을 나누어 사용한다.
그리고 [12 factors app 에서 가이드](https://12factor.net/ko/config)하고 있는 바는 설정 값은 코드에 상수로 저장하기보다는 설정을 코드에서 엄격히 분리하라고 가이드 한다.

서버 환경에서는 동적으로 환경 변수를 주입해 줄 수 있다.
Nodejs 에서는 일반적으로 NODE_ENV 라는 변수에 development, staging, production 과 같은 값을 넣고,
[node-config](https://www.npmjs.com/package/config) 와 같은 라이브러를 사용하여 런타임에 적절한 환경 설정 값을 사용할 수 있게 한다.

그러나 React, Vue 와 같은 SPA (Single Page Application) 를 작성하면 설정 값의 주입이 정적 시간, 즉 빌드 시간에 이루어지기 때문에 서버 환경에서와 같은 방법을 사용할 수 없다.
이를 해결하기 위한 방법으로 다음과 같은 방법이 있다.

(본 포스팅에서는 AWS S3 와 같은 정적 호스팅을 이용하지 않고 nginx 환경에서 배포하는 환경에서의 방법을 논한다.)

1. 환경 별로 서로 다른 빌드를 만든다.
2. Nginx 와 같은 웹 서버 실행 시 스크립트를 통해 환경 변수를 넣어준다.

# 환경 별로 서로 다른 빌드를 만든다.

빌드 시에 개발용 환경 변수, QA 용 환경 변수, 상용 환경 변수를 선택하여 빌드하고 배포하는 방식이다.

개발용으로 빌드할 때에는 아래와 같이 실행하고,

```shell
$ NODE_ENV=development yarn build
```

상용으로 빌드할 때에는 아래와 같이 실행한다.

```shell
$ NODE_ENV=production yarn build
```

그 결과로 나온 빌드 파일을 nginx 와 함께 배포한다.

단순하고 직관적이지만 환경에 따라 앱 자체가 달라지는 것이 아님에도 불구하고 매번 빌드를 해줘야 한다는 단점이 있다.

# Nginx 와 같은 웹 서버 실행 시 스크립트를 통해 환경 변수를 넣어준다.

`Docker`, `React` 그리고 Nginx 로 구성된 예시를 살펴보자.

React 에서의 예를 살펴보자.
React 에서는 기본적으로 dotenv 를 사용한다.
`.env`, `.env.development`, `.env.production` 이 각각 기본값, 개발용, 상용 환경 변수 값을 담고 있다.

nginx 실행 시, 주입되어 있는 `NODE_ENV` 의 값과 환경 변수에 따라 환경 설정 값을 담은 js 파일을 생성하여
index.html` 에 주입시켜주는 방식이다.

## dotenv 를 파싱하여 js 파일 만들어주기

다음과 같이 `env.sh` 파일을 작성해준다.

```shell script
#!/bin/bash

# NOTE: You need to install bash version 4 or above.
# This script makes use of associative arrays.
# https://stackoverflow.com/questions/1494178/how-to-define-hash-tables-in-bash

set -e

declare -A key_values
readonly CURRENT_DIR=$(dirname "$0")
readonly DIST="$CURRENT_DIR"/env-config.js

function trim() {
    local var="$*"
    # remove leading whitespace characters
    var="${var#"${var%%[![:space:]]*}"}"
    # remove trailing whitespace characters
    var="${var%"${var##*[![:space:]]}"}"
    printf '%s' "$var"
}

function read_env() {
  local env_filename=$1

  # Read each line in .env file
  # Each line represents key=value pairs
  while read -r line || [[ -n "$line" ]];
  do
    line=$(trim $line)

    # Ignore empty string
    if [ -z "$line" ]; then
      continue
    fi

    # Ignore comments
    first_letter=$(echo $line | head -c 1)
    if [ "$first_letter" = '#' ]; then
      continue
    fi

    # Split env variables by character `=`
    if printf '%s\n' "$line" | grep -q -e '='; then
      varname=$(printf '%s\n' "$line" | sed -e 's/=.*//')
      varvalue=$(printf '%s\n' "$line" | sed -e 's/^[^=]*=//')
    fi

    key_values[$varname]=${varvalue}
  done < "$CURRENT_DIR/$env_filename"
}

function read_env_from_environment_variables() {
  # Read value of current variable if exists as Environment variable
  # If value does not exist as Environment variable, use current value from above.
  for key in "${!key_values[@]}"; do
    value=$(printf '%s\n' "${!key}")
    # If value
    [[ -z $value ]] && value=${key_values[$key]}
    key_values[$key]=$value
  done
}

# Recreate config file
rm -f "$DIST"
touch "$DIST"

read_env ".env"
read_env ".env.${NODE_ENV}"
read_env_from_environment_variables

# Add assignment
echo "window._env_ = {" >> "$DIST"

# Append configuration property to JS file
for key in "${!key_values[@]}"; do
  echo "  $key: '${key_values[$key]}'," >> "$DIST"
done

# Add the closing brace
echo "}" >> "$DIST"

```

이 스크립트는 `.env` 와 `.env.{NODE_ENV}` 그리고 shell 에 설정된 환경 변수 값을 읽어서 `env-config.js` 에서 자바스크립트 객체로 만들어준다.

## `env-config.js` 를 `index.html` 에 주입하기

React 기본 구조 상에서 `public/index.html` 의 헤더 부분에 다음과 같이 `env-config.js` 를 넣어주는 태그를 추가한다.

```html
<script src="%PUBLIC_URL%/env-config.js"></script>
```

## Dockerfile 작성하기

Nginx 를 사용하여 배포하는 것은 [다른 포스팅](https://dev.to/subhransu/nevertheless-subhransu-maharana-coded-5eam)을 참고하여 작성하고, 추가적으로 `env.sh` 를 Docker 컨테이너에 추가하고, nginx 실행 전에 실행시켜 `env-config.js` 파일이 생성되도록 한다.
그리고 아래의 설정을 Dockerfile 에 추가해준다.

```yaml
COPY ./env.sh .
COPY .env .
COPY .env.development .
COPY .env.production .

RUN apk add --no-cache bash

RUN chmod +x env.sh

CMD ["/bin/bash", "-c", "/usr/share/nginx/html/env.sh && nginx -g \"daemon off;\""]
```

## 실행하기

이제 런타임에 환경 설정 값을 변경할 준비가 되었다.

개발용으로 실행시킬 때는 아래와 같이 실행할 수 있다.

```shell
$ docker run -it -p 80:80 -e NODE_ENV=development {DOCKER_IMAGE}
```

상용으로 실행시킬 때는 아래와 같이 실행시킬 수 있다.

```shell
$ docker run -it -p 80:80 -e NODE_ENV=production {DOCKER_IMAGE}
```

# 참고

- https://www.freecodecamp.org/news/how-to-implement-runtime-environment-variables-with-create-react-app-docker-and-nginx-7f9d42a91d70/
