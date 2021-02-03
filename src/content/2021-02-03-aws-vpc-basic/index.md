---
draft: false
title: AWS VPC 기본
author: [Woojin]
date: 2021-02-03
image: ./aws_vpc.png
imageAlt: AWS VPC
tags: ['AWS', 'VPC', 'Basic']
---

# VPC

__V__irtual __P__rivate __C__loud

클라우드 상에서 논리적으로 공간을 격리하기 위해 사용한다.

VPC 가 없다면 EC2 인스턴스들이 서로 혹은 인터넷과 스파게티처럼 연결될 것이다. 이렇게 되면 시스템의 복잡도가 엄청나게 올라간다.
최악의 경우, 하나의 인스턴스를 추가하는 데에 거의 모든 인스턴스를 수정해야 하는 상황이 발생할 수 있다.
EC2 인스턴스를 VPC 로 묶고, VPC 별로 네트워크 설정을 해준다면, 이 복잡도를 완화시킬 수 있다.

## VPC in AWS

region 별로 복수의 VPC 를 생성할 수 있는데, 최대 5개까지 생성 가능한다. (이 limit 은 AWS 에 요청하여 늘릴 수 있다.)

CIDR 는 VPC 당 최대 5개 이며, 각각의 CIDR 는 아래와 같은 제한을 갖는다.
- 최소 서브넷 크기는 /28 = 16 개의 IP
- 최대 서브넷 크기는 /16 = 65,536 개의 IP

VPC 는 private 이기 때문에, 오직 private IP 대역만 허용한다.

VPC CIDR 는 가지고 있는 다른 네트워크의 CIDR 와 겹쳐서는 안된다.

CIDR 와 public, private IP 에 대한 설명은 [지난 포스팅](/2021-02-02-cidr-public-private-ip/index/) 을 참고한다.

## Default VPC (기본 VPC)

AWS 에서는 새로 계정을 생성하면 region 별로 기본 VPC 를 생성하여 제공한다.

기본 VPC 는 인터넷 연결을 가지고 있고, 모든 인스턴스는 public IP 를 갖게 된다.

또한, public, private DNS 이름도 갖게 된다.

## Subnets in VPC

VPC 의 subnet 은 AZ(Availability Zone) 에 종속된다. VPC 내에 다수의 subnet 을 생성할 수 있다.
이 때, subnet 의 CIDR 는 VPC 의 CIDR 범위 안에 있어야 한다.

public subnet 은 인터넷에 연결된 subnet 을 말하고, private subnet 은 인터넷에 연결되지 않은 subnet 을 말한다.

일반적으로 public subnet 은 private subnet 에 비해 작은 CIDR 를 갖는다.

AWS 는 각 subnet 별로 5개의 IP 주소를 예약한다. (첫 4개 + 마지막 1개)

예: 10.0.0.0/24 CIDR 블록에 대해
- 10.0.0.0: 네트워크 주소
- 10.0.0.1: VPC 라우터를 위해 예약
- 10.0.0.2: 아마존에서 제공하는 DNS 매핑을 위해 예약
- 10.0.0.3: 향후 사용을 위해 예약
- 10.0.0.255: 네트워크 브로드캐스트 주소 (AWS 는 VPC 내 브로드캐스트를 지원하지는 않는다. 단순히 예약되어있어 사용은 불가.)

## Internet Gateway (IGW)

IGW 는 VPC 인스턴스가 인터넷에 연결될 수 있게 한다.
IGW 를 설정하지 않으면, public IP 를 할당 받더라도 EC2 인스턴스는 인터넷에 연결되지 않는다.

IGW 는 VPC 와 분리되어 생성된다.

VPC 는 오직 하나의 IGW 에 attach 될 수 있으며, IGW 역시 하나의 VPC 를 attach 할 수 있다.

IGW 자체로는 인터넷 접근을 허용할 수는 없으며, Route table 이 같이 편집되어야 한다.

## NAT 인스턴스

NAT: __N__etowrk __A__ddress __T__ranslation

(Outdated 되어 사용을 권하진 않음, [참고](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)) 

NAT 인스턴스는 private subnet 을 인터넷에 연결할 수 있게 한다.

public subnet 에 런치되어야 한다.

EC2 의 Source/Destination Check 플래그를 disable 해야 한다.

Elastic IP 가 attach 되어 있어야 한다.

Route table 이 private subnet 으로부터 NAT 인스턴스로 route 하도록 설정되어 있어야 한다.

즉, public subnet 에 인스턴스를 하나 띄워서 private subnet 의 인스턴스가 그 인스턴스를 통해 인터넷에 접근하도록 만드는 것이다.

NAT 인스턴스는 Community AMI 에서 검색하여 사용하면 된다. 

### 단점

- 인스턴스의 유형에 따라 성능의 차이를 보일 수 있다. (micro 와 같이 작은 인스턴스 사용시에는 네트워크 성능이 저하될 수 있음)
- 새로운 포트의 개방이 필요할때마다 NAT 인스턴스의 security group 을 수정해주어야 한다.

## NAT Gateway

NAT gateway 는 NAT 인스턴스의 대안으로 사용하면 된다. NAT 인스턴스의 기능을 AWS 에서 managed 서비스로 제공하는 것이다.

지정된 AZ 에 Elastic IP 를 사용하여 NAT gateway 를 생성하게 된다.

NAT Gateway 가 설정된 subnet 안의 인스턴스는 사용이 불가하고 다른 subnet 의 인스턴스만 사용이 가능하다.

NAT Gateway 는 Internet Gateway 를 필요로 한다. (Private Subnet => NAT => IGW)

45Gbps 까지 bandwidth 가 자동적으로 스케일업이 가능하다.

Security Group 을 생성하고 관리할 필요가 없다.

NAT 인스턴스와 NAT Gateway 의 비교는 [이 문서](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-nat-comparison.html) 를 보면 자세히 나와 있다.

## Network ACLs (NACL)

NACL 은 subnet 에 적용되는 firewall 이라 생각하면 된다.
하나의 subnet 당 하나의 NACL 이 있으며, 새로 생성된 subnet 에서는 Default NACL 이 할당된다.
Default NACL 은 모든 outbound, inbound 트래픽을 허용한다.
단, 새로 임의로 생성한 NACL 은 모든 outbound, inbound 트래픽을 deny 하도록 생성된다. 

NACL 은 다음과 같은 규칙을 정의한다.
- 규칙은 1 ~ 32766 의 숫자를 갖고, 낮은 숫자일수록 우선순위가 높다.
- 마지막 규칙은 asterisk(*) 이고 다른 모든 규칙이 매칭되지 않을 때의 행동을 정의한다.
- AWS 에서는 규칙별로 100단위로 숫자를 할당하는 것을 권장한다. (추후 중간에 규칙을 끼워넣을 수 있도록)

NACL 은 subnet 수준에서 특정한 IP 를 블록하는 좋은 수단이다. 

### Network ACLs vs Security Group (SG)

NACL 은 subnet 단위, SG 는 EC2 인스턴스 단위의 규칙이라는 점에서 가장 크게 차이점이 있다.

NACL 은 allow, deny 규칙을 지원하지만, SG 는 allow 규칙만 지원한다.

또한 NACL 은 stateless 지만, SG 는 stateful 하다.
(stateless: 규칙에 명시되어 있는대로만 행동함, stateful: return 트래픽에 대해서는 규칙에 상관없이 허용됨)

이 외의 NACL 와 SG 의 차이점은 [이 문서](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html) 를 참조하도록 하자.
