## 결국 es 는 로깅 + 조회 용일뿐
- 데이터 가공한걸 넣고 쓰는용으로는 맞지않다.
- 가공은 다른곳에 저장하고, 순정 log로 보고 편하게 쌓고 보고싶을 떄 유용하다.

## 별 생각없으면 적용하면 좋은 부분
- index 는 가능하면 시간, 날짜 기반으로 설정할 수 있도록 하고, life cycle management 기능을 이용해 적당한 날짜가 지나면 삭제해준다.
- **같은 데이터 포맷이면 같은 index로 쌓일 수 있도록 한다.** => 이런 이유로 모든 nginx 로그 하나로 통합하는게 이득
- [index-template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html) 은 무조건 설정해서 효율적으로 index를 관리하자.
- 트래픽이 적은 index의 경우 hot 노드를 처음부터 쓰지않고 warm 으로 들어가게 하면 hot node shard 가 well balanced 해지기 떄문에 좋다.
  - 이는 다 섞어서 쓰게 되는경우, num of shards 를 비슷하게 가져가려고하는 elastic search rebalance 정책상 특정 노드에 트래픽이 몰리는 상황이 올 수 있다.
  - 트래픽 많은 index 들은 shard 숫자를 적절히 셋팅해 사용 node를 묶고, 적은 노드들은 warm으로 보내 서로 독립적으로 돌도록해버린다.

## es는 RESTful Api를 지원하기 떄문에 언제 어디서든 조회가 너무나도 강력하다.
- {endpoint}/{원하는index-pattern}/_search 로 어디서든 데이터 조회가 가능하다.
- spring-data-elastic 의 RestClient.java 로 권장되긴 하지만, 복잡한 설정 필요없이 일반 RestTemplate.java 를 사용해도 아주 편하게 조회해올 수 있다.
```
	String url = UriComponentsBuilder.fromHttpUrl(ES_ENDPOINT)
					.path("/contents-news-*")
					.path("/_search").toUriString();
	HttpEntity<String> entity = new HttpEntity<>(ES_REQUEST_BODY, makeElasticSearchRequestHeaders());
	return new RestTemplate().postForEntity(url, entity, ESNewsAccessResponse.class);
```

## cpu usage에 영향을 주는 항목들
1. 높은 트래픽
	- 트래픽이 많아서 indexing 을 자주 하는 경우 cpu 가 튄다.
2. 적은양의 memory size
	- elasticsearch 는 자바로 개발되었고, mem size가 적으면 gc가 자주 일어나기 떄문에 cpu 사용률이 높아진다.
이런경우 memory usage도 같이 확인해서 문제시 노드 스펙을 늘려본다.
3. 너무 많은 양의 shard 숫자
	- shard 숫자가 많으면 조회시 그만큼 뒤져야할 cpu 일감이 많아져서 느려짐.
	- 그럼 적정 숫자는 얼마인지는 운영하면서 파악을 해봐야하는데..

## 샤드 갯수 정리
- 하나의 샤드는 가능하면 50g를 넘기지 않도록 구성한다.
- 클러스터 전체 샤드가 1000개 이상 되면 맛이 간다고 봐도 될 것 같다.
- 트래픽이 많은 index의 경우, node가 분산해서 데이터 쌓을 수 있게, shard 갯수 = cpu 코어 숫자 * 노드 수 (replica 0 일때) 
- ex) 멀티쓰레드지원 쿼드 코어로 5개 노드가 주어지면,  2 * 4 * 5 = 40개가 이론상 빠르나.. index가 많아지면 조회시 cpu usage 가 그만큼 올라간다.
