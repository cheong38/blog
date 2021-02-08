---
draft: false
title: AWS S3
author: [Woojin]
date: 2021-02-06
image: ./main.png
imageAlt: S3 logo
tags: ['AWS', 'S3']
---

# S3

S3 는 __object__ (파일) 를 __bucket__ (디렉토리) 에 저장할 수 있게 한다.

bucket 이름은 전역적으로 유일해야 한다. (다른 사용자가 사용중인 이름은 사용불가)

bucket 은 region 수준으로 정의된다.

## 이름 컨벤션
- 대문자 사용 불가
- _(underscore) 사용 불가
- 3 ~ 36 글자 사이의 길이
- IP 주소 형식 불가
- 소문자나 숫자로 시작해야 함

## Object

Object (파일) 은 __key__ 를 가지고 있다.

key 는 bucket 하위의 전체 경로를 의미한다.

예:
- s3://my_bucket/__my_file.txt__
- s3://my_bucket/__my_folder/another_folder/my_file.txt__

key 는 _prefix_ + __object_name__ 으로 구성된다.

예: 
- s3://my_bucket/_my_folder/another_folder/___my_file.txt__

bucket 안에는 디렉토리 개념이 없지만 UI 적으로는 디렉토리 같은 형식으로 보여준다.
실제로는 단순히 / 를 포함하는 key 가 긴 object 일 뿐이다.

Object 의 __value__ 는 body 의 내용을 말한다.
- 최대 Object 크기는 5TB 이다.
- 그러나 한 번에 5GB 이상을 업로드할 수 없으므로 그 이상의 object 를 업로드하기 위해서는 multi-part upload 를 사용해야 한다.

__Metadata__ 는 key/value 쌍으로 이루어진 텍스트의 리스트이다.

__Tag__ 는 최대 10개까지 가질 수 있고, unicode 로된 key/value 쌍이다. 보안이나 라이프사이클을 위해 유용하다.

__Version ID__ 는 versioning 이 enable 인 경우에 할당되는 versioning 에 대해서는 뒤에 설명 예정이다. 

## S3 Versioning

S3 에서는 파일들에 대해 __버전 관리__를 할 수 있다.
이 버전 관리는 __bucket 수준__ 으로 enable, disable 된다.

같은 key 로 업로드되는 파일은 1, 2, 3... 과 같이 __증가하는 순서__ 로 버전 관리가 된다. (숫자 형식으로 version ID 가 생성되진 않는다.)

bucket 에 대해 버전관리를 하는 것이 __Best Practice__ 이다.
- 의도되지 않은 삭제로부터 파일을 지킬 수 있고,
- 이전 버전으로 쉽게 롤백할 수 있다.

versioning 을 enable 하기 전에 업로드된 파일은 version 이 __null__ 기록 기록된다.

bucket 단위로 versioning 을 __enable__ 하거나 __suspend__ 할 수 있다.
enable 은 versioning 자체를 활성화하는 것이고,
suspend 는 모든 작업에 대해 버전 생성을 일시 중단하지만, 기존 object 의 버전은 유지한다.

versioning 되고 있는 bucket 에서 object 를 삭제하면, 실제로 삭제되는 것이 아니라 object 에 delete marker 생성하게 된다.
이 delete marker 를 삭제하면, 삭제된 object 를 복구할 수 있다.
