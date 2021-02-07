---
draft: true
title: AWS EC2 User Data 핸즈온
author: [Woojin]
date: 2021-02-08
image: ./main.jpg
imageAlt: EC2 icon
tags: ['AWS', 'EC2', 'User Data']
---

# EC2 User Data

EC2 User Data 스크립트를 이용해 인스턴스를 __bootstrap__ 하는 것이 가능하다.

bootstrap 이란 머신이 시작할 때의 시작 커맨드들을 의미한다.

이 스크립트는 인스턴스의 첫 시작 때 __딱 한 번__ 실행된다.

EC2 User Data 는 다음과 같은 작업들을 자동화할 때 사용된다.
- 소프트웨어 설치하기
- 인터넷에서 공통 파일들 다운로드하기
- 그 외 인스턴스를 실행할 때 필요한 그 어떤 것

EC2 User Data 는 __root__ 권한으로 실행된다.

## 핸즈온

이번 핸즈온에서는 nginx 로 User Data 로 다음과 같은 작업을 수행한다.
- nginx 를 설치
- nginx 기본 페이지를 간단한 Hello World 페이지로 대체

---- 실제 AWS 콘솔에서의 실행화면 캡처해서 순차적으로 핸즈온 넣을 것
