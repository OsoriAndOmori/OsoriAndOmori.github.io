---
title: Spring-data-rest 간단한 정리
author: OsoriAndOmori
date: 2022-06-27 18:00:00 +0900
categories: [Blogging, Spring]
tags: [spring, spring-data, spring-data-rest]
---

### spring-boot-starter-data-rest
- spring-data-...가 하도 많아서 매우 햇갈림.
- `spring-mvc` 기반
  - `reactive` 지원 의지 없음. [이슈 링크](https://github.com/spring-projects/spring-data-rest/issues/1299#issuecomment-752913911)
- import 하고 repository 에 annotation `@RepositoryRestResource` 붙이면 controller 부터 entity 기준으로 CRUD 메서드가 생긴다.
- `swagger` 도 간편하게 붙일 수 있음.
```groovy
//Swagger
implementation 'io.springfox:springfox-swagger2:3.0.0'
implementation 'io.springfox:springfox-data-rest:3.0.0'
implementation 'io.springfox:springfox-swagger-ui:3.0.0'
```
```java
@EnableSwagger2
@Import(SpringDataRestConfiguration.class) //swagger 설정임.
@SpringBootApplication
public class AdminApplication extends WebMvcConfigurationSupport {

	public static void main(String[] args) {
		SpringApplication.run(AdminApplication.class, args);
	}

	@Bean
	public Docket api() {
		return new Docket(DocumentationType.SWAGGER_2)
				.select()
				.apis(RequestHandlerSelectors.any())
				.paths(PathSelectors.any())
				.build();
	}

	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/swagger-ui/**").addResourceLocations("classpath:/META-INF/resources/webjars/springfox-swagger-ui/");
		registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/springfox-swagger-ui/");
	}
}

```
- `spring-mvc` 기반이기 떄문에 reactive 와 같이 쓸수 없음.
- `@RepositoryRestResource` 전처리 후처리를 위해 EventHandler `@RepositoryEventHandler` 를 제공함.
```java
@Slf4j
@Component
@RepositoryEventHandler
public class RadioChannelsEventHandler {
    @HandleBeforeCreate
    public void handleCreate(RadioChannels radioChannels) {
        log.info("START Radio Channel Create");
    }

    @HandleBeforeSave
    public void handleSave(RadioChannels radioChannels) {
        log.info("START Radio Channel Save");
    }
}

```
- property 설정으로 base-path 지정 가능
```groovy
spring.data.rest.base-path: /api
```
- rest 응답결과에 default 로 `@Id` 가 노출이 안됨
  - 설정으로 entity 마다 등록해줘야함.
```java
@Configuration
public class RestConfiguration implements RepositoryRestConfigurer {

    /**
     * 응답 결과에 @Id 노출을 위한 설정. 노출을 위해선 노출할 클래스 하나하나 등록해줘야함.
     */
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config, CorsRegistry cors) {
        RepositoryRestConfigurer.super.configureRepositoryRestConfiguration(config, cors);
        config.exposeIdsFor(RadioChannels.class);
        config.exposeIdsFor(RadioRecommendedChannels.class);
        config.exposeIdsFor(RadioUserLikedChannels.class);
    }
}
```
