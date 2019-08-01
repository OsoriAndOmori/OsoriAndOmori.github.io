### 1. 우리가 늘 쓰는버릇대로 (엔터칠 때 기분 좋음)
```java
@Autowired
private XXService xXService;
```

### 2. 생성자 (스프링에서 권장하는 방식, 코드도 집어넣을 수 있음)
```java
public class MovieRecommender {
    private final CustomerPreferenceDao customerPreferenceDao;
    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }
}
```

### 3. 롬복 (우린 Autowired 외 멤버변수도 많아서 못써먹을수도)
![58667484-8f33-11e8-95ac-2aab5c8f9035](https://user-images.githubusercontent.com/22016317/48462554-f3fceb80-e81b-11e8-8c7c-dea120a81fbf.jpeg)
