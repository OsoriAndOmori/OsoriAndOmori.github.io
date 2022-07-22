---
title: Spring batch JdbcCursorReader.java 사용시 확인할 점
author: OsoriAndOmori
date: 2022-07-21 18:00:00 +0900
categories: [Blogging, Spring]
tags: [spring, spring-batch]
---

잘 아시는분이 있을것 같아서 질문드립니다.
스프링 배치에서 쓰는 jdbcCursorReader 는
코드상 쿼리 조회해서 온 jdbc ResultSet 을 기반으로 하나씩 데이터를 읽어내는 reader 인데요..
제가 분석하기론 ResultSet  내 쿼리의 결과물을 다 들고 하나씩 next next 하며 read 로 보내줘서 데이터 읽어 내는것 같습니다. (실제로 조회해온 ResultSet 에서 브레이크 걸어보면 전체 데이터가 다 들어있습니다.)
저는 그래서 커서리더는 최초 메모리에 전체를 올려야하기 떄문에
몇백만건의 많은양의 데이터를 처리하는데는 부적절하다고 생각했고,  메모리 이슈도 있다고 생각했는데요
좀 유명한 스타개발자 아저씨가 쓴 아티클에 의하면, streaming 으로 db에서 빨아오는 형태라고해서
뭔가 전체데이터를 들고있을 이유가 없는것 같은데, 뭐가 맞는지 모르겠네용..
혹시 정확히 아시는분이 있다면 알려주시면 감사하겠습니다.
7-3. CursorItemReader
위에서 언급한대로 CursorItemReader는 Paging과 다르게 Streaming 으로 데이터를 처리합니다.
쉽게 생각하시면 Database와 어플리케이션 사이에 통로를 하나 연결하고 하나씩 빨아들인다고 생각하시면 됩니다.
JSP나 Servlet으로 게시판을 작성해보신 분들은 ResultSet을 사용해서 next()로 하나씩 데이터를 가져왔던 것을 기억하시면 됩니다.
https://jojoldu.tistory.com/336

두분께서 언급해주신 부분들은 cursor reader 의 특징들 기반
거의 맞는것 같고,,,
참고로 아까 오전에 더 연구해봤더니… 정확하게 알았습니다.
그 궁금증의 시작은 cursorReader fetchSize 를 20으로했는데
최초 read 시 왜 응답값 전체를 들고있냐였거든요.
(쓰레드 시작 이미지 참고)
집중해서본건 계속 JdbcCursorReader.java 고
지금 저희 배치 상태는 jdbc  Cursor Reader 써도
fetchSize 만큼 db 로부터 계속 받아오는게 아닌
조회해야할 데이터 db 한방에 가져오도록 되어있습니다. ㅎㅎ
(단우님께서 언급주신 [실제 데이터 < fetchSize] 문제는 아닙니다
그리고.. mybatis cursor reader썼어도 똑같이 안될것 같습니다. 같은 드라이버를 써서).
mysql 드라이버단까지 디버깅해서 원인 찾아냈는데,
driver 단 connection URL 설정에 useCursorFetch=true 설정을 추가해야:땀_흘리는_웃는_얼굴: mysql driver 내 쿼리 실행준비를 담당하는 statement.setFetchSize() 가 동작을 합니다…
https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-implementation-notes.html
(cursor reader에서 builder 로 옵션 받은걸 여기다 넣음)
그리고 statment 가 실행이 될 떄 cursorFetch 를 하고, 응답받는건 ResultsetRowsCursor 라는거로 받게됩니다.
그러면 응답 결과 ResultSet 가져올 때, cursor 기반으로 데이터를 가져오고 ResultSet.next() 하면서 하나씩 정보를 저장하는데,, 부족하면 db 가서 더 가져오는 식의 메모리 절감이 됩니다.
예를들어 테이블에 200만개 데이터여도 memory엔 제가 지정한 fetchSize 10 만큼만 들고 reader 에서 하나씩 chunk 를 쌓고, 들고있는걸 다 소모하면 db 가서 더 없나 한번 기웃거린다음에 가져오는 행위를 반복합니다.
ResultSet.next() 실행시 이 로직이 포함되어있습니다…
<- 이러면 커서가 제대로 동작한다고 볼수가 있죠
useCursorFetch=true connection option 을 적용하지 않고
cursor reader 를 쓰는 방법은 약속된 숫자 -2^31
reader.fetchSize(Integer.MIN_VALUE) 인 경우에만 커서리더를 동작시킬 수 있습니다. 요사이즈로 들어오면 driver 가 streaming 형태로 응답을 내려줘서 말그래도 next 하면서 하나씩 streaming 할 수 있습니다.
처음부터 데이터를 전부 fetch 해오는걸 디버깅하다 눈으로 보니깐
처음에 혼란스러웠던것 같고  + 같이 찾아주신부분들까지
유의미하게 정리해서 공유해볼게요~
이런게 제대로 적혀있는 블로그도없습니다.
jdbc cursor reader 주석도 형편없고…
커서기반 리더는 계속 가져와야해서 네트워크 통신도 많이 쓰고
커넥션도 길게잡고, 트랜잭션도 길게 써야하는 단점이 있으나
아주 큰 데이터를 저사양 어플리케이션으로 배치처리할 때 잘 쓸 것 같습니다. (편집됨)
