---
title: Spring Security 기술 간단 요약 (~ing)
author: OsoriAndOmori
date: 2022-07-26 14:00:00 +0900
categories: [Blogging, Spring]
tags: [spring, spring-mvc, spring-security]
---

> 이 게시물은 [백기선 - Spring Security](https://www.inflearn.com/course/백기선-스프링-시큐리티) 요약임 <br>
> 모든 코드는 [OsoriAndOmori/playground-spring](https://github.com/OsoriAndOmori/playground-spring/tree/main/applicaion-web-mvc) 에 올려둠. <br>
> 멋대로 만든 프로젝트 구조 이기 떄문에, 강좌 내용은 따라가나 구조는 절대 따라가지 않음..

1. 스프링 시큐리티 연동
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

