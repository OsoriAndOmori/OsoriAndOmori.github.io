- jdbc orm framework
- 배치에 써봄
- spring-data-jpa 와 미묘하게 다름. 더 심플하다고 볼 수 있음. jpa 엔 lazy execution 이나 캐싱 같은것을 안에서 다 해주는데 이녀석은 즉시 실행하고, 성능적으론 jpa 보다 아쉬운 부분이 있으나 Simple 함.
- spring data jdbc 는 아직 delete 에 관해서는 derived Query 를 제공하지 않음
entity 방식으로 지우는게 아니면 @Query @Modify 써서 native 쿼리를 실행해야함.
배치는 orm 보단 쿼리를 사용하는게 효과적인것 같음..... jdbcTemplate 이나 쓰자
