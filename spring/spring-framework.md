## [스프링 프레임워크 레퍼런스](https://docs.spring.io/spring-framework/docs/current/reference/html/index.html)

## IoC : Inversion of Control
- 제어의 역전
- 객체의 생성과 소멸 제어를 원래는 작성자가 함. `new Instance`
- 벗 스프링에서 `특수한 Annotation` 을 붙이거나, 직접 `@Bean` 등록된 객체들은 IoC Container = ApplicationContext 가 구동시 다 만들어 들고 있음.
- 최초 Spring 구동시 등록하는 로깅이 잘 보임. 
- 그래서 너무 많은 Bean 을 만드는 상황이면 최초 구동속도가 느려짐...
- 이렇게 만들어둔 Bean 들은 DI로 사용처에 주입이 된다.

## DI : Dependency Injection
- IoC 와 긴밀하게 연결되어있는 특징
- 객체의 생성과 소멸을 IoC Container 가 해주는 것이고, 생성된 객체는 주입해서 사용
- 주입해서 사용한다는 말은, new 해서 새로 만들지 않는다는 것.
- `DI` 한 class 들 간 hash 값 비교하면 동일한것을 알 수 있음
`Assert.equals(this, ApplicationContext.getBean("myService"))`

## AOP : Aspect Oriented Programming
- 관점 지향형 프로그래밍. 비즈니스 로직에 집중을 하고, 그 외에는 별도의 방법으로 로직 살을 붙이자.
- 공통된 반복로직을 제거하는데 용이하게 쓸 수 있음.
- AOP 를 구현하는 방법은 3가지정도 원래 있음
  - 컴파일 단계 : .class 만들 때, 조작해서 로직 집어넣음 
  - 바이트코드 조작단계 : .class 파일 만들고 메모리에 올릴때 조작도 가능
  - [프록시 패턴](https://refactoring.guru/design-patterns/proxy) : Spring 에서 요걸 씀. 정상적으로 빌드하고 패턴을 이용해 프록시 AOP 도입
    - 프록시 패턴에 대해 적지 않을 수 없음.
    - 기존 코드를 건드리지 않고 새 로직을 추가하기
    - [java code example](https://refactoring.guru/design-patterns/proxy/java/example)
    - 예를 보면 Proxy객체는 공통 interface 상속을 하고, 원래 것을 멤버변수로 들고 있음.
      - 실행시 Proxy 객체를 주입해서 사용하면 추가된 로직 + 원래 로직 그대로 실행할 수 있어서 동작 추가 가능.
      - 리펙토링에 적극 활용해도 좋음.
- `@Aspect` 샘플코드는 널리고 깔렸으니 생략

## PSA : Portable Service Abstraction
- 추상화를 통해 
- Controller
- transactionManager
- CacheManager

## [예제로 배우는 스프링 입문](https://www.inflearn.com/course/spring_revised_edition/dashboard)
- 나는 어디까지 알고 싶은건지부터 정해야함
- 대충 뭔지는 암 -> 중요한 개념들은 알고 개발은 개념 다 써서 할 수 있음 -> [가성비 안나오는 벽] -> **좀 깊게 알아서 컨트리뷰션 할 수 있는 정도** -> [넘사벽] -> 저자들 수준으로 다알아
- 가성비 안나오는 곳 까지만 파보자



