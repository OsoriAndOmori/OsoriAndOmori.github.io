---
title: Spring Security 요약 (~ing)
author: OsoriAndOmori
date: 2022-07-26 14:00:00 +0900
categories: [Blogging, Spring]
tags: [spring, spring-mvc, spring-security]
---

> 이 게시물은 [백기선 - Spring Security](https://www.inflearn.com/course/백기선-스프링-시큐리티) 요약임 <br>
> 모든 코드는 [OsoriAndOmori/playground-spring](https://github.com/OsoriAndOmori/playground-spring/tree/main/applicaion-web-mvc) 에 올려둠. <br>
> 멋대로 만든 프로젝트 구조 이기 떄문에, 강좌 내용은 따라가나 구조는 절대 따라가지 않음..

## 1. 스프링 시큐리티 연동
- gradle dependency 추가

```gradle
    implementation 'org.springframework.boot:spring-boot-starter-security'
```

- 추가를 하게 되면 기본적으로 모든페이지는 `인증(Authentication)` 을 하게됨.
- 로그인 페이지도 기본적으로 제공이 됨.
- 스프링 구동시 로컬 환경에서 쓸 수 있는 기본 비밀번호가 주어짐. 아이디는 `user`

```console
2022-07-26 14:33:05.315  WARN 14922 --- [           main] .s.s.UserDetailsServiceAutoConfiguration :
Using generated security password: f93ee0c0-f58b-4f39-ab94-7805c203d2d5
This generated password is for development use only. Your security configuration must be updated before running your application in production.
```

- 해결해야할 숙제
  - 인증없이 접근 가능한 URL을 설정하고 싶다.
  - 이 애플리케이션을 사용할 수 있는 유저 계정이 그럼 하나 뿐인가?
  - 비밀번호가 로그에 남는다고?

## 2. 인증없이 접근 가능한 URL 설정
- SecurityFilterChain Bean 등록하면 끝임.
- chaning 형태로 `and()` 활용하여 예쁘게 쓸 수도 있고 그냥 끊어서 해도 상관없음.
- 아래 예제는 `/`, `/info` 는 모두에게 접근 허용하고, `/admin` 의 경우에는 `ADMIN` role 을 가지고 있는 자만이 접근 가능함.

```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .mvcMatchers("/", "/info").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().permitAll();
        http.formLogin(); //폼로그인
        http.httpBasic(); //http 기본 로그인

        return http.build();
    }
```

## 3. jpa & database 연동
- jpa 쓰고 싶으니깐 jpa dependency 추가 하고 테이블이랑, repository 하나 만들어줌

```groovy
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

```java
public interface AccountRepository extends JpaRepository<Account, Integer> {
    Account findByUsername(String username);
}

@Entity
@Data
public class Account {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Integer id;
  @Column(unique = true)
  private String username;
  private String password;
  private String role;

  //spring security 가 원하는 기본적인 비밀번호 포맷 {알고리즘}
  public void encodePassword() {
    setPassword("{noop}" + getPassword());
  }
}
```
- db 연동시 핵심은 `UserDetailsService` 를 구현한 Bean 이 있어야함. 등록만 하면 바로 적용됨.
- **여기까지 새로운 문제**
  - `{noop}` 을 없앨 수는 없을까?
  - 테스트는 매번 화면키고 해야 하는건가?

## 4. Password Encoder 적용
- spring 5 이전엔 따로 PasswordEncoder 없이, 그냥 문자 그대로 저장했었음. (아님 사용자가 직접 해시 알고리즘 써서 비번 바꾸거나겠지?)
- 왜 맨 앞에`{noop}`포맷이 생겼을까라고 한다면, 4.x 대 까진 NoOpPasswordEncoder 가 기본이었는데 버전을 올리면 Bcrypt 로 변경됨
  - 그러다보니 버전업을하면 인증이 전부 꺠지게됨... 그래서 앞에 prefix 를 통해, 여지를 주기위해 포맷을 저렇게 잡았음.
- 아래 bean 을 추가해주고 최초 유저 집어넣을 떄 비번 encoding 한번 해주자.
- 그러면 기본 bcrypt 로 셋팅 되어서 비밀번호가 저장됨. 원하면 포맷을 {MD5}도 변경할 수 있음.
- 로그인시 PasswordEncoder bean 이용해서 `인증` 실시함.
```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    public void encodePassword(PasswordEncoder passwordEncoder) {
        this.password = passwordEncoder.encode(this.password);
    }
```
![image](https://user-images.githubusercontent.com/22016317/182273294-7c55c673-0f53-4f16-90e2-a4d6cd05c02c.png)

- **여기까지 새로운 문제**
  - 테스트는 매번 화면키고 해야 하는건가?
