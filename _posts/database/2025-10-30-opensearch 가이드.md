---
title: Opensearch 가이드 용 문서
author: OsoriAndOmori
date: 2025-10-30 12:00:00 +0900
categories: [Blogging, Database]
tags: [database, elasticsearch, sharding, opensearch]
---

1.️ 분산형 검색 및 분석 엔진
- Elasticsearch 오픈소스 기반으로 Amazon이 유지 중인 Community-Driven Fork
- 분산 인덱싱 + 검색 엔진 → 대용량 로그, 메트릭, 문서 데이터를 수평 확장으로 처리
- REST API 기반으로 쉽게 접근 (/_search, /_cat/indices 등)

2. 인덱스(Index)와 샤드(Shard) 구조
- 인덱스 = 데이터베이스의 테이블 개념
- 샤드 = 인덱스의 물리적 분할 단위 (기본 Primary + Replica 구성)
- 노드 단위로 자동 분산 저장되어 장애 대응 및 부하 분산 가능
- `_cat/shards` 명령으로 샤드 상태 확인 가능

3. 인프라 구성
- master(대장), coordinate(트래픽받는애), data node(데이터 저장 노드)
- nginx 외부망 열린 -> k8s ingress helm -> dashboard pod

4. 인덱스 관리 (dashboard 에서)
- index management
- policy
- template
- alias

5. Alerting (알림 기능)
- 조건 기반 트리거 설정 가능 (CPU > 80%, 에러로그 100건 이상 등)
- Slack, Email, Webhook 등으로 통보 가능

----------
- [같이 보면 좋은 정리](https://osoriandomori.github.io/posts/ElasticSearch-1%EB%85%84%EA%B0%84-%EC%9A%B4%EC%98%81%ED%95%98%EB%A9%B0/)
