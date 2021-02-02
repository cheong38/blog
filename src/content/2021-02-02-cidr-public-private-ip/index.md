---
draft: false
title: CIDR, Public/Private IP
author: [Woojin]
date: 2021-02-02
image: ./ip-address.png
imageAlt: IP Address Logo
tags: ['cidr', 'IP', 'public IP', 'private IP']
---

# 개요

인터넷에서는 통신한 컴퓨터를 찾기 위해 IP 주소를 사용한다.
이번 포스팅에서는 IP 주소의 범위를 표기하는 방식인 CIDR 과 public, private IP 개념에 대해 다룬다.

# CIDR

__C__lassless __I__nter-__D__omain __R__outing

CIDR 은 IP 의 범위를 정의하는데 사용된다.

> WW.XX.YY.ZZ/MM

위와 같은 형태로 표기된다.

여기서 `WW.XX.YY.ZZ` 는 흔히 우리가 사용하는 IP 주소이며, `MM` 은 0 ~ 32 사이의 정수이다.

`WW.XX.YY.ZZ` 부분을 __base IP__ 라고 하고 `/MM` 부분을 __subnet mask__ 라고 한다.

base IP 는 IP 범위 안에 포함되는 IP 하는 표현하고,
subnet mask 는 IP 주소 내에 얼마나 많은 비트가 변할 수 있는지를 표현한다.

## Subnet mask 이해하기

| Subnet mask | IP 개수 |
|-------|------|
|/32|2^0 = 1|
|/31|2^1 = 2|
|/30|2^2 = 4|
|/29|2^3 = 8|
|/28|2^4 = 16|
|/24|2^8 = 256|
|/16|2^16 = 65,536|
|/0|all|

자주 사용하는 값은 아래와 같다.

| Subnet mask | 설명 |
|-------|------|
|/32|어떤 IP 숫자도 변경될 수 없다.|
|/24|마지막 IP 숫자 한개가 변경될 수 있다.|
|/16|마지막 IP 숫자 2개가 변경될 수 있다.|
|/8|마지막 IP 숫자 3개가 변경될 수 있다.|
|/0|모든 IP 숫자가 변경될 수 있다.|

### 연습

- 192.168.0.0/24

=> _192.168.0.0 ~ 192.168.0.255 (256 개의 IP)_

- 10.1.0.0/16

=> _10.1.0.0 ~ 10.1.255.255 (65,536 개의 IP)_

- 172.168.0.1/32

=> _172.168.0.1 오직 한 개_

- 0.0.0.0/0

=> _모든 IP_

계산을 확실히 확인하고 싶다면, [ipaddressguide 웹사이트](https://www.ipaddressguide.com/cidr) 를 확인하면 좋다.

# Public vs Private IP (IPv4)

IANA(Internet Assigned Numbers Authority) 는 IP 주소를 public 과 private 구역으로
나누어서 설정하였다.

- Private IP 구역

| IP 범위 | CIDR | 사용 |
|------|-----|-----|
| 10.0.0.0 ~ 10.255.255.255 | (10.0.0.0/8) | 큰 네트워크에 사용 |
| 172.16.0.0 ~ 172.31.255.255 | (172.16.0.0/12) | AWS 의 기본 private IP 로 사용 |
| 192.168.0.0 ~ 192.168.255.255 | (192.168.0.0/16) | 홈 네트워크에 사용 |

- Public IP 구역

Private IP 구역을 제외한 모든 구역이 Public IP 구역이다.
