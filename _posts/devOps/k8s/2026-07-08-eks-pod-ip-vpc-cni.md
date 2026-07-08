---
title: EKS에서는 Pod도 Subnet IP를 먹는다
author: OsoriAndOmori
date: 2026-07-08 08:00:00 +0900
categories: [Blogging, DevOps]
tags: [k8s, eks, aws, vpc-cni, subnet]
---

Kubernetes 를 처음 배울 때는 보통 이렇게 이해한다.

> Pod 마다 고유한 IP 는 필요하지만, 그 IP 가 꼭 실제 VPC/Subnet IP 일 필요는 없다.

실제로 일반적인 Kubernetes 환경에서는 이 말이 맞다.

예를 들어 로컬 Mac 에 Kubernetes 를 설치해도 여러 Pod 를 띄우고 서로 통신시킬 수 있다.
내 Mac 이 AWS VPC 안에 있는 것도 아니고, Pod 마다 실제 Subnet IP 를 받은 것도 아닌데 통신은 된다.

왜냐하면 Calico, Cilium Overlay, Flannel 같은 일반적인 CNI 환경에서는 Pod 네트워크를 Kubernetes 내부의 Overlay 네트워크로 만들 수 있기 때문이다.

대충 이런 느낌이다.

```text
실제 네트워크
└── Node IP 만 실제 네트워크 IP 사용

Kubernetes 내부 네트워크
├── Pod A: 10.x.x.x
├── Pod B: 10.x.x.x
└── Pod C: 10.x.x.x
```

이 경우 Pod IP 는 클러스터 안에서 의미 있는 가상 IP 에 가깝다.
밖으로 직접 나갈 주소가 아니라면 굳이 실제 VPC/Subnet IP 일 필요가 없다.

그래서 나도 처음에는 EKS 도 당연히 이 방식이라고 생각했다.

> Node 만 Subnet IP 를 쓰고, Pod 는 내부 가상 IP 를 쓰겠지?

그렇게 생각하면 실제로 소모되는 Subnet IP 는 Node 수 정도라고 볼 수 있다.
Pod 가 아무리 많아져도 Overlay 네트워크 안에서만 IP 를 쓰니까 Subnet IP 가 크게 줄어들지 않을 거라고 봤다.

그런데 EKS 기본 네트워크는 방식이 다르다.

## EKS 기본 네트워크는 AWS VPC CNI 를 쓴다

EKS 에서 기본으로 사용하는 네트워크는 AWS VPC CNI 다.

AWS VPC CNI 는 일반적인 Overlay CNI 처럼 Pod IP 를 클러스터 내부 가상 네트워크에서만 관리하지 않는다.
Pod 에도 VPC 의 private IP 를 할당한다.

즉, 구조가 이렇게 된다.

```text
Node: VPC/Subnet IP 사용
Pod : VPC/Subnet IP 사용
```

공식 문서에도 Amazon VPC CNI add-on 이 EC2 Node 에 ENI 를 붙이고,
각 Pod 에 VPC 의 private IPv4 또는 IPv6 주소를 할당한다고 설명되어 있다.

그러니까 Pod 수가 많아지면 Subnet IP 도 같이 줄어든다.

내가 착각했던 지점은 여기였다.

```text
일반 Kubernetes Overlay CNI
└── Subnet IP 는 Node 수만큼 주로 소모된다고 생각해도 됨

EKS 기본 AWS VPC CNI
└── Node + Pod 가 모두 Subnet IP 를 소모함
```

## 왜 이렇게 IP 를 많이 쓰는 구조일까?

처음 보면 좀 아깝다.

> 굳이 Pod 마다 실제 VPC IP 를 줘야 하나?

그런데 AWS 입장에서는 이 방식의 장점이 있다.

Overlay 네트워크를 한 겹 덜 쓰기 때문에 성능과 지연시간 측면에서 유리하고,
Pod 가 VPC 네트워크 위의 일급 리소스처럼 동작할 수 있다.

예를 들어 이런 것들이 자연스러워진다.

- VPC 라우팅과 직접 연동
- Security Group, NACL, Route Table 같은 AWS 네트워크 모델과의 일관성
- AWS Load Balancer 와의 연동
- VPC 안의 다른 AWS 리소스와 직접 통신
- 네트워크 트러블슈팅 시 AWS 네트워크 관점으로 추적 가능

즉 EKS 기본 네트워크는 Kubernetes 안에 별도 가상 네트워크를 강하게 만드는 쪽보다,
Kubernetes Pod 를 AWS VPC 네트워크 안에 자연스럽게 편입시키는 쪽을 선택한 것이다.

대신 대가는 명확하다.

> Pod 수만큼 VPC/Subnet IP 를 소모한다.

## EKS Auto Mode 에서 더 헷갈렸던 부분

EKS Auto Mode 를 쓰면 Node 를 직접 EC2 목록에서 관리하는 느낌이 덜하다.
AWS 가 Node 운영을 많이 숨겨주기 때문이다.

그래서 처음에는 이런 식으로 오해하기 쉽다.

> Node 가 안 보이니 내 Subnet IP 도 별로 안 쓰는 거 아닌가?

하지만 Node 가 안 보인다고 해서 Node 가 없는 것은 아니다.
내가 직접 관리하지 않을 뿐, 실제로는 내 VPC/Subnet 안에 Node 와 Pod 가 배치된다.

대충 이런 구조로 보면 된다.

```text
내 AWS 계정
└── VPC
    ├── Private Subnet A
    │   ├── Auto Mode Node
    │   ├── Pod A
    │   └── Pod B
    └── Private Subnet B
        ├── Auto Mode Node
        ├── Pod C
        └── Pod D
```

Auto Mode 에서는 AWS 가 networking, IP addressing, network interface 관리까지 많이 자동화한다.
특히 Pod networking 에 prefix delegation 같은 방식을 사용해서 IP 자원을 관리한다.

하지만 큰 방향은 같다.

Pod 는 여전히 VPC/Subnet IP 자원을 사용한다.

## 그래서 Subnet CIDR 크기가 중요하다

일반적인 Overlay CNI 관점으로 EKS 를 설계하면 Subnet 을 너무 작게 잡을 수 있다.

예를 들어 "Node 몇 개 안 쓸 건데 /28 이면 되지 않나?" 같은 생각을 할 수 있다.
하지만 EKS 기본 네트워크에서는 Pod 수도 같이 봐야 한다.

```text
필요한 IP 수
= EKS control plane 이 만드는 ENI
+ Node 에 필요한 IP
+ Pod 에 필요한 IP
+ Load Balancer 등에 필요한 IP
+ 여유분
```

AWS 문서상 클러스터 생성/업데이트에 지정하는 Subnet 은 최소 사용 가능한 IP 수 조건이 있고,
운영 환경에서는 그보다 훨씬 넉넉하게 잡는 것이 좋다.

특히 Pod 가 많이 뜨는 서비스라면 Subnet IP 부족은 바로 장애로 이어질 수 있다.
Pod 가 스케줄링되어도 IP 를 할당받지 못하면 정상적으로 올라오지 못한다.

## 정리

일반 Kubernetes 라고 항상 Pod 가 실제 VPC/Subnet IP 를 쓰는 것은 아니다.

Overlay CNI 를 쓰는 환경에서는 Node 만 실제 네트워크 IP 를 쓰고,
Pod 는 클러스터 내부의 가상 네트워크 IP 를 사용할 수 있다.

반면 EKS 기본 네트워크인 AWS VPC CNI 에서는 Pod 도 VPC/Subnet IP 를 사용한다.

그래서 EKS 에서는 클러스터 설계 시 Subnet CIDR 을 넉넉하게 잡아야 한다.
Node 수만 보고 IP 를 계산하면 안 되고, 실제로 뜰 Pod 수까지 같이 계산해야 한다.

내가 이번에 다시 정리하면서 얻은 결론은 이거다.

```text
일반 Kubernetes Overlay CNI
└── Node 만 실제 VPC/Subnet IP 를 주로 사용

EKS 기본 AWS VPC CNI
└── Node 와 Pod 모두 실제 VPC/Subnet IP 사용
```

즉 EKS 에서는 Pod 도 Subnet IP 를 먹는다.

## 참고

- [Assign IPs to Pods with the Amazon VPC CNI](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html)
- [Learn about VPC Networking and Load Balancing in EKS Auto Mode](https://docs.aws.amazon.com/eks/latest/userguide/auto-networking.html)
- [View Amazon EKS networking requirements for VPC and subnets](https://docs.aws.amazon.com/eks/latest/userguide/network-reqs.html)
