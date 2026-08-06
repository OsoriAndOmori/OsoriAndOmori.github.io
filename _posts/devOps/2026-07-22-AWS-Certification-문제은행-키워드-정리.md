---
title: AWS Solutions Architect Associate 문제은행 키워드 정리
author: OsoriAndOmori
date: 2026-07-22 12:00:00 +0900
categories: [Blogging, DevOps]
tags: [aws, cloud, saa-c03, certification]
toc: true
---

AWS Solutions Architect Associate(SAA-C03) 문제은행을 풀다가 헷갈린 개념을 짧게 정리한다.

IAM, EC2, S3 같은 기본 설명은 생략한다. 문제 보기에서 **비슷한 서비스 중 무엇을 골라야 하는지**, **특정 요구사항이 어떤 서비스의 힌트인지** 떠올리는 용도다.

시험 범위는 [AWS 공식 SAA-C03 Exam Guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html)를 기준으로 한다.

## 네트워크와 전송

### Gateway Endpoint

S3와 DynamoDB를 VPC 안에서 인터넷이나 NAT Gateway 없이 접근할 때 사용한다. 라우팅 테이블에 경로가 추가되며 별도 시간당 비용이 없다.

### Interface Endpoint (AWS PrivateLink)

서브넷에 사설 IP를 가진 ENI를 만들고 지원되는 AWS 서비스에 비공개로 접근한다. Security Group을 붙일 수 있고 시간당 비용과 데이터 처리 비용이 발생한다.

### VPC Peering의 비전이성 (Non-transitive)

A-B, B-C가 Peering 되어 있어도 A-C는 통신하지 못한다. VPC가 많거나 중앙 허브 구조가 필요하면 Transit Gateway를 본다.

### AWS Transit Gateway

여러 VPC와 VPN을 중앙 허브에 연결한다. 대규모 연결이나 전이 라우팅은 편하지만, VPC 두 개만 단순 연결한다면 Peering보다 비쌀 수 있다.

### Route 53 Resolver Inbound Endpoint

온프레미스 DNS가 AWS VPC의 Private Hosted Zone 이름을 질의할 때 사용한다. 방향은 **온프레미스 → AWS DNS**다.

### Route 53 Resolver Outbound Endpoint

VPC의 자원이 온프레미스 도메인을 질의할 때 사용한다. Forwarding Rule과 함께 쓰며 방향은 **AWS → 온프레미스 DNS**다.

### AWS Global Accelerator

고정 Anycast IP를 제공하고 AWS 글로벌 네트워크를 통해 TCP/UDP 트래픽을 리전 엔드포인트로 전달한다. 콘텐츠 캐싱이 목적이면 CloudFront, 고정 IP·빠른 글로벌 네트워크 진입이 목적이면 Global Accelerator다.

### AWS PrivateLink

상대 VPC의 전체 네트워크를 연결하지 않고 특정 서비스만 사설로 노출한다. CIDR 중복이 있어도 사용할 수 있으며 서비스 제공 측에는 보통 NLB가 필요하다.

### Gateway Load Balancer

방화벽, IDS/IPS 같은 가상 네트워크 장비를 투명하게 트래픽 경로에 넣을 때 사용한다. 일반 웹 요청 분산용 ALB와는 목적이 다르다.

### Direct Connect Gateway

하나의 Direct Connect 연결로 여러 리전의 VPC 또는 Transit Gateway에 접근할 때 사용한다. Direct Connect 자체는 암호화를 제공하지 않으므로 필요하면 VPN을 함께 고려한다.

### Security Group vs Network ACL

Security Group은 ENI/인스턴스 수준의 상태 저장형 방화벽이라 허용 규칙만 작성한다. Network ACL은 서브넷 수준의 상태 비저장형 방화벽이라 허용·거부 규칙과 요청·응답 양방향을 각각 설정한다.

## 컴퓨팅과 확장

### Cluster Placement Group

인스턴스를 한 AZ의 가까운 하드웨어에 배치해 낮은 지연과 높은 네트워크 처리량을 얻는다. 대신 동시에 많은 인스턴스를 시작하기 어렵고 장애 범위가 커질 수 있다.

### Partition Placement Group

인스턴스 그룹을 서로 다른 하드웨어 파티션에 분산한다. HDFS, Kafka, Cassandra처럼 일부 노드 장애를 견디는 대규모 분산 시스템에 어울린다.

### Spread Placement Group

소수의 중요 인스턴스를 서로 다른 하드웨어에 최대한 분산한다. 함께 장애 나면 안 되는 작은 규모의 워크로드에 어울린다.

### EC2 Capacity Reservation

특정 AZ의 EC2 용량을 확보한다. 용량 보장이 목적이며 할인 상품은 아니다. 비용 할인은 Reserved Instance나 Savings Plans와 구분한다.

### Zonal Reserved Instance

특정 AZ에 적용하며 할인과 함께 용량 예약 효과가 있다. Regional RI는 더 유연하게 할인을 적용하지만 용량을 예약하지 않는다.

### EC2 Fleet / Spot Fleet

여러 인스턴스 타입과 AZ를 묶어 목표 용량을 확보한다. Spot 중단 위험을 줄이려면 단일 타입보다 여러 풀과 `capacity-optimized` 전략을 사용한다.

### Auto Scaling Lifecycle Hook

인스턴스 시작이나 종료를 잠시 멈추고 초기화, 로그 수집 같은 작업을 수행한다. 단순히 지표가 안정될 때까지 기다리는 Warm-up과는 다르다.

### Auto Scaling Warm Pool

미리 초기화된 인스턴스를 정지 또는 실행 상태로 보관해 Scale-out 시간을 줄인다. 부팅과 애플리케이션 준비가 오래 걸리는 경우에 유용하다.

### Reserved Concurrency vs Provisioned Concurrency

Lambda Reserved Concurrency는 함수가 사용할 수 있는 동시 실행 수를 보장하면서 상한도 건다. Provisioned Concurrency는 미리 실행 환경을 준비해 Cold Start를 줄인다.

### SQS Visibility Timeout과 Lambda

Lambda가 SQS 메시지를 처리하는 동안 다른 소비자에게 메시지를 숨기는 시간이다. 함수 처리 시간보다 너무 짧으면 같은 메시지가 다시 처리될 수 있으므로 멱등성도 필요하다.

## 스토리지와 데이터 이동

### S3 Object Lock

WORM 방식으로 객체 버전을 정해진 기간 삭제·변경하지 못하게 한다. Governance Mode는 특별 권한으로 우회 가능하고, Compliance Mode는 루트 사용자도 보존 기간 전 삭제할 수 없다.

### S3 Legal Hold

보존 종료일 없이 객체 버전을 보호한다. Retention Period와 별개로 동작하며, 권한이 있는 사용자가 해제할 때까지 유지된다.

### S3 Replication Time Control (RTC)

리전 간 또는 동일 리전 복제에 예측 가능한 복제 시간을 요구할 때 사용한다. 대부분의 새 객체를 15분 안에 복제하는 SLA와 모니터링을 제공한다.

### S3 Batch Replication

Replication Rule을 만들기 전에 존재하던 객체나 복제에 실패한 객체를 일괄 복제한다. 일반 복제 규칙은 기본적으로 새로 들어오는 객체를 대상으로 한다.

### S3 Multi-Region Access Points

여러 리전의 S3 버킷 앞에 하나의 글로벌 엔드포인트를 제공하고 가장 가까운 활성 리전으로 요청을 보낸다. 리전 장애 시 Failover Control로 트래픽을 전환할 수 있다.

### S3 Access Point

하나의 S3 버킷을 여러 애플리케이션이나 팀이 사용할 때 각각 별도 엔드포인트와 정책을 만든다. 거대한 Bucket Policy 하나를 계속 수정하지 않고 접근 경로별 권한을 분리할 때 사용한다.

### EBS Fast Snapshot Restore

스냅샷으로 만든 새 볼륨의 모든 블록을 처음부터 최대 성능으로 읽게 한다. 켜 둔 AZ마다 비용이 발생하므로 복구 직전에 자동 활성화하는 문제가 자주 나온다.

### EBS Multi-Attach

지원되는 io1/io2 볼륨 하나를 같은 AZ의 Nitro 기반 EC2 최대 16대에 연결한다. 애플리케이션이 동시 쓰기를 직접 조정해야 하며 XFS, EXT4 같은 일반 파일 시스템을 여러 서버가 동시에 쓰는 용도는 아니다.

### EFS vs EBS

여러 EC2가 NFS 파일 시스템을 동시에 공유하고 사용량에 따라 저장 용량이 자동 증감해야 하면 EFS다. EC2에 낮은 지연의 블록 디스크를 붙이는 문제라면 EBS이며, 여러 인스턴스 공유는 일반 기능이 아니라 Multi-Attach의 제한 조건을 확인해야 한다.

### FSx for Lustre

HPC, 머신러닝처럼 병렬 처리량이 중요한 리눅스 워크로드용 파일 시스템이다. S3와 연결해 데이터를 빠르게 처리하고 결과를 다시 S3에 쓸 수 있다.

### FSx for NetApp ONTAP

NFS, SMB, iSCSI와 ONTAP 기능이 필요한 경우 사용한다. 온프레미스 NetApp 워크로드를 익숙한 방식으로 AWS에 옮긴다는 표현이 힌트다.

### DataSync

온프레미스와 AWS 스토리지 또는 AWS 스토리지끼리 대량 데이터를 온라인으로 반복 전송한다. 예약 전송, 검증, 암호화가 필요하면 단순 CLI 복사보다 적합하다.

### 데이터 이전 서비스 고르기

온라인 대량 복사는 DataSync, 온프레미스 스토리지를 AWS와 계속 연동하면 Storage Gateway, 안정적인 상시 전용 회선은 Direct Connect다. Snowball Edge는 기존 고객의 오프라인 이전에는 쓸 수 있지만 신규 고객은 주문할 수 없으므로, 최신 문제에서는 AWS Data Transfer Terminal이나 파트너 솔루션도 확인한다.

### S3 File Gateway

온프레미스 애플리케이션에는 NFS/SMB 파일 공유처럼 보이지만 데이터는 S3 객체로 저장된다. 자주 쓰는 데이터는 로컬 캐시에 남는다.

### Volume Gateway: Cached vs Stored

Cached Volume은 주 데이터를 S3에 두고 자주 쓰는 데이터만 로컬에 캐시한다. Stored Volume은 전체 주 데이터를 온프레미스에 두고 AWS에 비동기 백업한다.

### Tape Gateway

기존 백업 프로그램의 가상 테이프 라이브러리(VTL) 인터페이스를 유지하면서 테이프 데이터를 S3 Glacier 계열에 보관한다. 물리 테이프 교체 문제가 나오면 후보로 본다.

### AWS Transfer Family

SFTP, FTPS, FTP, AS2 클라이언트를 유지하면서 백엔드를 S3나 EFS로 바꿀 때 사용한다. 대량 데이터 마이그레이션 자체가 목적이면 DataSync와 구분한다.

### S3 Lifecycle 전환

접근 패턴이 명확하면 `S3 Standard → Standard-IA → Glacier Deep Archive → Expiration` 순서로 자동 전환·삭제할 수 있다. Standard-IA는 생성 후 최소 30일이 지나야 전환할 수 있고, Deep Archive는 180일 최소 보관 비용이 있다는 점도 같이 본다.

## 데이터베이스와 캐시

### Aurora Reader Endpoint

여러 Aurora Replica로 읽기 연결을 분산하는 엔드포인트다. Writer 장애 조치용 Cluster Endpoint와 구분한다.

### Aurora Global Database

하나의 Primary 리전에서 여러 Secondary 리전으로 낮은 지연으로 복제한다. 글로벌 읽기와 리전 재해 복구가 목적이며, 여러 리전에서 동시에 쓰는 구조는 아니다.

### Aurora Serverless v2

트래픽에 맞춰 Aurora 용량을 세밀하게 자동 조절한다. 간헐적이거나 변동이 큰 관계형 DB 워크로드에서 인스턴스 크기를 고정하고 싶지 않을 때 본다.

### RDS Proxy

DB 연결을 Pooling하고 재사용해 Lambda처럼 짧은 연결이 폭증할 때 RDS/Aurora의 연결 부담을 줄인다. 읽기 결과 캐싱 서비스는 아니다.

### DynamoDB Global Tables

여러 리전에서 로컬처럼 읽고 쓸 수 있는 Multi-Region, Multi-Active 복제를 제공한다. 리전 간 충돌은 마지막 쓰기 우선 방식으로 해결한다.

### DynamoDB DAX

DynamoDB 전용 인메모리 캐시로 읽기 지연을 마이크로초 단위까지 줄인다. 애플리케이션에서 DAX 클라이언트를 사용해야 하며 강력한 일관성이 필요한 읽기에는 맞지 않는다.

### DynamoDB Adaptive Capacity

트래픽이 특정 파티션 키에 몰릴 때 파티션별 처리량을 자동 조정한다. 그렇다고 한 개의 극단적인 Hot Key 설계를 해결해 주는 것은 아니다.

### DynamoDB Provisioned vs On-demand

트래픽이 안정적이고 예측 가능하며 처리량을 직접 관리해 비용을 최적화하면 Provisioned Capacity를 본다. 트래픽이 불규칙하거나 급증하고 용량 계획 없이 요청당 과금하려면 On-demand가 맞지만, 이전 최고치의 두 배를 갑자기 넘는 급증은 Throttling 가능성을 고려한다.

### ElastiCache Redis vs Memcached

Valkey/Redis OSS는 복제, Multi-AZ, 자동 장애 조치, 백업·복원, 영속성, 복잡한 자료구조가 필요할 때 선택한다. Memcached는 복제나 영속성 없이 노드 장애 시 데이터가 사라져도 되는 단순 멀티스레드 분산 캐시에 어울린다.

## 메시징과 이벤트

### SQS FIFO Message Group ID

같은 Message Group 안에서만 순서를 보장한다. 서로 다른 Group ID를 사용하면 FIFO Queue에서도 여러 메시지를 병렬 처리할 수 있다. Message Deduplication ID나 Content-based Deduplication은 5분의 중복 제거 구간에 적용되며, 소비자 처리까지 무조건 한 번만 실행된다는 뜻은 아니므로 멱등성은 여전히 챙긴다.

### SQS Standard와 멱등성

Standard Queue는 At-least-once 전달이라 같은 메시지가 다시 도착할 수 있고 순서도 보장하지 않는다. 메시지 처리 결과를 중복 적용하지 않도록 요청 ID나 비즈니스 키를 이용해 소비자를 멱등하게 만든다.

### SQS Dead-letter Queue Redrive

여러 번 처리에 실패한 메시지를 DLQ로 보내 격리하고, 원인을 해결한 뒤 원본 Queue로 다시 이동시킨다. DLQ의 보존 기간은 원본보다 길게 잡는 편이 안전하다.

### SNS Subscription Filter Policy

SNS Topic의 모든 메시지를 모든 구독자에게 보내지 않고 메시지 속성이나 본문을 기준으로 필요한 구독에만 전달한다.

### EventBridge Archive and Replay

Event Bus의 이벤트를 보관하고 특정 기간의 이벤트를 나중에 다시 재생한다. 장애 수정 후 이벤트를 재처리해야 한다는 요구에 맞는다.

### Kinesis Data Streams vs Amazon Data Firehose

Streams는 여러 소비자가 실시간으로 직접 처리하고 재생해야 할 때 사용한다. Firehose는 스트리밍 데이터를 S3, Redshift, OpenSearch 등으로 최소 운영 부담으로 전달할 때 사용한다.

### SQS vs Kinesis Data Streams / Amazon MSK

작업을 한 소비자에게 분배하고 생산자와 소비자를 느슨하게 결합하면 SQS다. 여러 소비자가 같은 이벤트를 독립적으로 읽고 순서·보존·재처리가 필요한 스트림이면 Kinesis Data Streams, Kafka 호환 생태계가 명시되면 Amazon MSK를 본다.

### API Gateway Private API

인터넷에 공개하지 않고 Interface VPC Endpoint를 통해서만 호출하는 REST API다. 리소스 정책으로 허용할 VPC나 VPC Endpoint를 제한한다.

## 보안과 거버넌스

### KMS Key Policy

KMS Key 접근 제어의 핵심 정책이다. IAM Policy에 Allow가 있어도 Key Policy가 계정의 IAM 권한 사용을 허용하지 않으면 접근하지 못할 수 있다.

### KMS Grant

AWS 서비스나 애플리케이션에 KMS Key 사용 권한을 동적으로 위임한다. 임시 또는 세밀한 위임이 필요하고 Key Policy를 계속 수정하고 싶지 않을 때 사용한다.

### STS External ID

외부 SaaS 업체가 여러 고객 계정의 Role을 Assume할 때 Confused Deputy 문제를 막는다. 제3자에게 Role ARN과 함께 고객별 External ID를 사용하게 한다.

### Secrets Manager vs Parameter Store

DB 자격 증명이나 API Key를 저장하면서 자동 교체, 교차 리전 복제, 세밀한 감사가 필요하면 Secrets Manager를 본다. Parameter Store는 `String`, `StringList`, KMS로 암호화하는 `SecureString`을 지원하지만 기본 자동 교체 기능은 없어서 일반 설정값이나 저렴한 암호화 저장에 어울린다.

### SSE-S3 vs SSE-KMS

별도 키 관리 요구 없이 S3 기본 저장 암호화만 필요하면 추가 비용 없는 SSE-S3를 쓴다. 키 정책으로 접근을 제어하거나 키 사용을 CloudTrail에서 감사하고 고객 관리 키·교차 계정 접근이 필요하면 SSE-KMS를 본다.

### AWS RAM (Resource Access Manager)

Organizations 안의 다른 계정과 Subnet, Transit Gateway, Route 53 Resolver Rule 같은 지원 자원을 공유한다. 자원을 각 계정에 중복 생성하지 않고 중앙 관리할 때 사용한다.

### AWS Backup Vault Lock

백업 보존 정책을 WORM 방식으로 강제해 랜섬웨어나 관리자 실수로 인한 삭제를 막는다. Compliance Mode는 유예 기간이 지나면 AWS도 잠금을 제거할 수 없다.

### GuardDuty vs Inspector vs Macie

GuardDuty는 계정과 네트워크의 위협 탐지, Inspector는 EC2·컨테이너 이미지·Lambda의 취약점 관리, Macie는 S3의 민감 데이터 발견이 핵심이다.

### WAF vs Shield Advanced vs Firewall Manager

WAF는 HTTP 요청을 검사해 IP·국가 기반 차단, SQL Injection, XSS 같은 웹 공격을 막는다. Shield Standard는 모든 계정에 기본 포함된 네트워크·전송 계층 DDoS 보호이고, Shield Advanced는 강화된 DDoS 대응과 비용 보호를 제공한다. Firewall Manager는 Organizations 여러 계정에 이 정책들을 중앙 배포한다.

### SCP와 Permission Boundary

SCP는 Organization의 계정 또는 OU 전체가 가질 수 있는 권한의 최대치를 제한한다. Permission Boundary는 특정 IAM User/Role이 가질 수 있는 권한의 최대치를 제한한다. 둘 다 권한을 직접 부여하지 않는다.

## 복구 전략과 비용

### RPO vs RTO

RPO는 장애 시 허용 가능한 **데이터 손실 시간**, RTO는 서비스를 다시 살릴 때까지 허용 가능한 **복구 시간**이다.

### Backup and Restore

평소에는 백업만 유지하다 장애 후 인프라를 복원한다. 가장 저렴하지만 RTO와 RPO가 가장 길다.

### Pilot Light

핵심 데이터 계층만 다른 리전에 항상 작게 실행하고 애플리케이션 서버는 장애 시 확장한다. Backup and Restore보다 빠르고 Warm Standby보다 저렴하다.

### Warm Standby

전체 시스템의 축소판을 다른 리전에 항상 실행하고 장애 시 확장한다. Pilot Light보다 빠르게 복구하지만 평상시 비용이 더 든다.

### Multi-Site Active/Active

여러 리전에서 전체 시스템을 동시에 서비스한다. RTO가 가장 짧지만 비용과 데이터 일관성 설계 난도가 가장 높다.

### Compute Savings Plans vs EC2 Instance Savings Plans

Compute Savings Plans는 EC2 인스턴스 패밀리·리전뿐 아니라 Fargate와 Lambda까지 유연하게 적용된다. EC2 Instance Savings Plans는 특정 리전과 인스턴스 패밀리를 약정하는 대신 할인 폭이 더 크다.

### On-demand vs Spot vs Savings Plans vs Dedicated Host

약정 없이 중단되면 안 되는 단기·불규칙 워크로드는 On-demand, 중단 가능한 배치 작업의 최대 비용 절감은 Spot이다. 지속적인 사용 금액을 1년 또는 3년 약정하면 Savings Plans, 물리 서버 격리나 소켓·코어 기반 라이선스가 필요하면 Dedicated Host를 본다.

---

> 이후 추가 형식: 기본 정의는 빼고, 문제의 결정 조건과 헷갈리는 서비스 차이만 2~3문장으로 적는다.
