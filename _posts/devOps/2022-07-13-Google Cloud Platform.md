---
title: Google Cloud Platform (정리 -ing)
author: OsoriAndOmori
date: 2022-07-13 18:00:00 +0900
categories: [Blogging, DevOps]
tags: [cloud, gcp, google-cloud-platform]
---

### 개요
- 전통적인 IT(온프레미스): 기업이 할거 다해야함 -> IaaS -> PaaS ->
- 그리고 SaaS : 구글클라우드 자체.

- 그래도 클라우드도 첼린지할 이슈가 많음.
  - 보안 : 제2 전문업체에 기업 정보가 다 올라감. 클라우드가 털리면 데이터 전부 유출.
  - 인력 자원이 별로 없다...

### GCP 의 특장점. 타 클라우드와 비교해볼때
  1. 편리한 메시징 서비스
    - GCP 는 Pub/Sub 으로 모든 내용 수행 가능
    - 공식 지원 pub/sub
  2. 최고의 머신러닝 서비스
    - 파이썬 자바몰라도 SQL 문으로 ML 모델 생성 가능
    - Speech toText
    - Vision AI 감정 텍스트분석
  3. 안전하고 유연한 네트워크
    - 자체 네트워크 라인.
    - 전역적인 VPC
  4. 저렴함
    - 확실히 저렴
    - 장기 사용자에게 일부 할인 지원
  5. 리전이 많음. 28개, 85개의 가용영역
    - 자체 전용 인터넷 회선으로 구성
    - 꽤 최근에 서울 Region 생김

### 권한
- Im 매니지먼트을 통해 실시함. 협업 용