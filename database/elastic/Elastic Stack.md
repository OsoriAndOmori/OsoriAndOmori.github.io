Elastic Search 
Kibana 
LogStash
Beats

이 네개를 합쳐서 Elastic Stack 이라고 부른다.
<img width="1822" alt="2018-12-11 6 38 20" src="https://user-images.githubusercontent.com/22016317/49791534-0a8f5780-fd74-11e8-806c-68d84edd90b1.png">

단순하게 생각해서,

Elastic Search - RESTful 로 요청 가능한 DB
Kibana - 전체적인 시스템 구경하는 GUI 툴
LogStash - 데이터 조작해서 Elastic에 넣는녀석
Beats - Simple 하게 데이터 Generate 하는 녀석

`Beats 가 쏘고 LogStash가 가공해서 Elastic Search에 넣고 Kibana 로 구경한다`

## 동영상 강좌
[Elastic Search](https://www.elastic.co/kr/webinars/getting-started-elasticsearch)
[Kibana](https://www.elastic.co/webinars/getting-started-kibana)
[LogStash](https://www.elastic.co/kr/webinars/getting-started-logstash)
[Beats](https://www.elastic.co/kr/webinars/using-beats-and-marvel-to-monitor-your-infrastructure)