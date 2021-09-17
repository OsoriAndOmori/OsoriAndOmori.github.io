# Varnish 
서버캐시, 로드밸런싱 기능있는 오픈소스

1. 특징
  - grace mode : 두 개 요청오면 하나만 보내고 대기, 백엔드 200 OK 아니면 캐시된 내용으로 서빙
  - 캐시를 메모리에 할 수도, file 로 할 수도 있음. malloc,2G만 사용해봄. 빠른응답위해 쓴거니깐.
  - saint mode : 백엔드 고장났으면, 캐시된 내용으로 서빙함
  - ttl 은 원하는대로 셋팅가능. 10초정도로 사용.
  - 로그밸런싱, 압축 지원 : 라운드로빈으로 하는게 일반적, 백엔드 응답을 압축해서 들고 있어 캐시 히트율 높일 수 있음.
  - 
2. 동작방식
  - vcl flow 에 따라 동작함
  - ![image](https://user-images.githubusercontent.com/22016317/133756092-8ed30d8a-f771-4385-8d15-8373e7f0e8fd.png)
  - vcl_recv: 웹 요청을 받으면 실행된다. 웹 요청을 변경하거나 캐시 사용 여부를 결정한다.
  - vcl_hash: hash 함수의 입력을 결정한다. 기본은 query string을 포함한 URL과 웹 서비스의 도메인 네임이다. 쿠키 값이나 User-Agent 값을 추가할 수 있다.
  - vcl_fetch: 원본 서버가 보낸 답장을 받으면 실행된다. 원본 서버가 보낸 답장을 변경하거나 TTL 값을 정한다.
  - vcl_deliver: 사용자에게 답장을 보내기 전에 실행된다. 사용자에게 보낼 답장을 변경할 수 있다.

3. 프로세스 구성
  - parent : 그냥 감시자
  - child : 실제 일하는 놈
    - acceptor(요청받음0, worker(vcl flow 따라 일함), epoll(처리된거 넘기는 쓰레드), expire(메모리 돌아다니면서 ttl 지난거 삭제), health-checker

4. 주의할점
  - 임시 저장 공간: Varnish에는 주 저장 공간과 임시 저장 공간이 있다. 주 저장 공간은 malloc, file, persistent 중의 하나로 지정된다. 주 저장 공간은 그 크기를 제한할 수 있지만, 임시 저장 공간은 크기 제한이 없다. TTL의 값이 shortlived 파라미터(기본값은 10초)보다 작거나 같으면 임시 저장 공간에 저장된다. 모든 데이터의 TTL을 일률적으로 10초 미만으로 설정하면 모든 데이터가 임시 저장 공간에 저장된다. 뉴스 서비스에서는 shortlived 파라미터를 변경하여 데이터가 임시 저장 공간에 저장되지 않게 했다.
  - keepalive: 특별한 경우를 제외하면 NHN의 웹 서버들은 keepalive-off로 운영한다. 반면 Varnish는 keepalive-on이 기본이다. 브라우저와 연결 지속 시간을 제어하는 파라미터로 sess_timeout이 있다. 이 파라미터가 지정하는 시간 이내에 요청이 없으면 브라우저와의 연결을 종료시킨다. 이 파라미터의 기본값은 5초이고 최솟값은 1초이다. Varnish가 keepalive-off로 동작하게 하는 방법은 두 가지가 있다. 첫 번째는 Varnish가 보내는 답장의 헤더에 "Connection: close"를 추가하여 보내는 것이다. 이 헤더를 받은 정상적인 브라우저는 Varnish와의 연결을 종료시킨다. 그러나 클라이언트가 "Connection: close"를 무시하는 프로그램이라면 연결이 종료되지 않는다. 두 번째 방법은 연결을 종료시키는 간단한 함수가 포함된 Varnish 모듈을 작성하는 것이다.
  - overflow: 트래픽이 폭주하는 경우 listen queue overflow가 발생할 수 있다. overflow가 발생하면, Linux 커널 파라미터인 net.core.somaxconn의 값과 Varnish의 파라미터인 listen_depth의 값을 조정하여 listen queue의 크기를 증가시킨다.

참고 : https://d2.naver.com/helloworld/352076
