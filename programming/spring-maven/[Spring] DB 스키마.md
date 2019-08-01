## Batch DB 구조
<img width="868" alt="스크린샷 2019-07-29 오후 5 20 21" src="https://user-images.githubusercontent.com/22016317/62032748-66382380-b225-11e9-947f-ade32450211a.png">

## Batch 기본
<img width="1003" alt="스크린샷 2019-07-29 오후 5 21 27" src="https://user-images.githubusercontent.com/22016317/62032749-66382380-b225-11e9-8684-b40e2e7349c1.png">

- step, job 등 bean 설정 한 것들은 당연히 id 가 겹쳐선 안됨. 나중것이 앞에것을 덮는 느낌. autowired 만 하다보면 헤딩할 수도 있음.

- job execution id : 실행시켰을 떄 생기는 아이디, 1개만 생김
- job instance id : job 호출을 받으면 instance 를 만들어서 실행하게 되는데, retry 를 할 수도 있으므로 하나의 job execution id 에 여러개 job instance id 가 생길 수도 있음.


- 참고 : https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/en/Ch02_SpringBatchArchitecture.html



