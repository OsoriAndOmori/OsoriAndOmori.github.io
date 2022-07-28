---
title: Real Mongo DB Summary (~ing)
author: OsoriAndOmori
date: 2022-07-21 18:00:00 +0900
categories: [Blogging, Book]
tags: [database, mongodb]
---

# 1. MongoDB
## 1.1 데이터베이스 트렌드
- 오라클 RDBMS, MS SQL Server 라이센스 비용 때문에 빡쳐서 -> 다들 MySql 로 Run (facebook, google, twitter)
- 구글은 Mysql 쓰다가 Bigtable(No SQL) + Spanner(분산 트랜잭션) 연구 시작
- 페북은 카산드라를 버리고 트위터 링크드인과 MySQL 을 발전시킨 -> WebScaleSQL 개발함
- 결국 여기까지 트렌드만 보면, '확장성'. MySQL 이 부족한 Scalability 를 다들 찾고 있음을 느낄 수 있음.
- Maria DB 도 2014년 이떄쯤 등장
- 2012년 쯤부터 카산드라 등 NoSQL 성장함. 그러나 결국 남은건 HBase (GFS, Bigtable 기반) + MongoDB
- 그러나 2000년대 후반까진 NoSQL DBMS 들이 기능 빵꾸가 너무 많아서 상용으로 쓰긴 힘들었음.
- 카산드라 + HBase 는 자바 기반 NoSQL 로 GC 로 인한 성능 저하가 늘 여전히 고민. 입지가 줄어든편
- Mongo
  - 트랜잭션지원, 분산처리, 재해복구, 샤딩 & 리밸런싱, 데이터복제 자동복구 지원
  - WiredTige 엔진 장착이후 부터 날개달기 시작함.

## 1.6 아키텍쳐
![image](https://user-images.githubusercontent.com/22016317/181588901-bd9375b2-7508-4d08-883e-ed4abc19305b.png)

# 2. Storage Engine
## 2.1 MMAPv1
- 이제 사실 거의 안씀. 초창기 몽고의 스토리지 엔진
- 자체 구현 캐시가 없고, os layer 의 캐시를 사용하는데 이는 system call 을 해야해서 느림
- os 자체 캐시시스템의 설정을 건드는 것 또한 사이드이펙트가 있을 수 있어서 적용 어려움
- 그래도 사실상 고대 몽고의 엔진이어서 다룬 느낌.
- 몽고설정에서 `engine: mmapv1` 입력시 적용됨
- 데이터 파일 구조, 데이터 파일당 사이즈 정도 설정하는게 있어서 성능 튜닝은 딱히 할 것이 없음.
