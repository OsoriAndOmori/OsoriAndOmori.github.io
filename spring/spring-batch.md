- chunk 단위 배치 프레임워크
  - job step tasklet 의 계층을 이루고 있음
  - tasklet 은 일반적으로 reader, processor(생략가능), writer 구조를 잡고
  - chunk 단위로 처리하며 commit 함. 롤백도 chunk 단위로 진행됨
  - reader 는 커서기반, 페이징 기반으로 구분되어있는데 별생각 없으면 paging reader 쓰면 

- spring batch test 에선 @Transactional  로 롤백이 안됨.
결론부터 얘기하면 spring-batch 는 실행되는 각각의 상태나 변수 등을 jobRepository 를 통해 DB 에 저장을 해둡니다.
이를 비즈니스 로직을 담당하는 transactionManager 와 묶어버리면 실행되는 상태나 변수 등이 한 번에 rollback 되거나 commit 되기에 rollback 이 안됩니다.
애초에 시작도 안됨.

- java -jar 로 바로 job 실행 가능하고, web 으로 띄워서 살행도 가능은하나 jar 실행을 권장함 (리소스 다르게, 서버 필요없고, 빌드서버에서 실행가능 등등 장점)
- StepScope 와 JobScope 개념.
  - 둘다 bean 을 prototype 으로 생성을 하는 것임. @StepScope를 tasklet reader 등 에 붙이는데 보통, 리턴값을 잘 설정하지 않으면 문제가 생김
  - https://jojoldu.tistory.com/132
