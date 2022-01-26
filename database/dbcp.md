## 메인DB mysql connection pool
- mysql 뿐만아니라 모든 db는 db쪽에서 `client connection pool`  설정을 갖는다.
- 사용자가 아무리 많은 connection 맺을라고해도, 결국엔 `client connection pool` 이상을 넘지 못한다.
- 넘는 경우 연결을 만들 수 없다며 에러 받고 db access 불가능함.
- `master` 기준 현재 `평시 6,500`, `피크치 7,000` 로 보임

## 근데
- 이것보단 dbcp 의 `maxActive` 수치가 적절히 셋팅되어있는지가 더 중요함.
- 컴포넌트는 보통 늘고 있고 점점 autoscale (scale out) 기술을 적용하니 높은 트래픽인 경우,  db connection 은 계속 증가함.
- 각각의 컴포넌트는 `어느 정도 수치` 로 제어할 필요는 있음. (물론 인프라 시스템상 알아서 늘어나긴하지만.. 개발자답게 안 된다고 전제하에)
  - `컴포넌트 dbcp maxActive 의 합 <= mysql connection Pool 설정`
  - 무분별한 코드 복붙보다는 작은 컴포넌트는 max Active 값을 줄여서 미리 장애방지

## 각각 컴포넌트의 톰캣 설정. 쓰레드 Pool 숫자와도 밸런스 맞추는것이 필요함.
- tomcat thread 숫자는 10인데, connection pool 이 50인건 의미가 없음
- `tomcat thread 숫자 >= dbcp` 가 바람직하나 그 차이가 큰 건 별로
- `tomcat thread(2048) > dbcp maxActive(180)`
