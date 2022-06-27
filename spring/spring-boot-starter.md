- spring 은 우리가 아는 DI, IoC, Pojo, AOP 특징을 가진 프레임워크
- spring-boot 는 spring 프로젝트 내 단일 application 으로 동작 할 수 있도록 편의를 갖춰준 프로젝트
- spring-boot-starter 는 어플리케이션 제작을 할 때 dependency 와 버저닝을 지원하기 위해 만든 boot 의 sub module. (이하 `starter`)
- `starter` 는 `AutoConfiguration` 의 지원으로 정말 대부분의 동작을 추상화 시켜버림
  - 이는 `@ConditionalOnClass` `@ConditionalOnBean` `@ConditionalOnWebApplication` `@ConditionalOnMissingBean` 등을 이용하여, boot 시작 때 설정값들을 선택적으로 bean 등록.
  - `@Bean` 으로 등록이 되면 사용자는 원하는 컴포넌트를 가이드대로 `@Autowired` 해서 사용하면 됨. 
- `spring-boot-starter` - `build.gradle` 을 보면 재밌음
  - 사실상 모든 starter 들에 다 들어 있는 고정멤버. `api(project(":spring-boot-project:spring-boot-starters:spring-boot-starter"))
  `
```groovy
plugins {
	id "org.springframework.boot.starter"
}

description = "Core starter, including auto-configuration support, logging and YAML"

dependencies {
	api(project(":spring-boot-project:spring-boot"))
	api(project(":spring-boot-project:spring-boot-autoconfigure"))
	api(project(":spring-boot-project:spring-boot-starters:spring-boot-starter-logging"))
	api("jakarta.annotation:jakarta.annotation-api")
	api("org.springframework:spring-core")
	api("org.yaml:snakeyaml")
}
```
- `spring-boot`
  - `SpringApplication.java` 가 들어있는 boot 의 main
- `autoconfigure`
  - 모든 starter 의 설정 META-INF 들고 있는 모듈의 bean 을 등록해준다.