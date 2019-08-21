Spring 사용시 외부로 `http` 요청할 때 [RestTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) 을 많이 쓰는 편 입니다.
혹은 apache commons의 `HttpClient` 도 있는것 같구요.
그러나 요것들은 기본적으로 사용하자면 [Sync & blocking](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/) 라
요청이 날아가면 응답이 올 때까지 다른 작업을 못하고 대기를 칩니다.

대량의 요청을 처리속도가 오래걸리는 서버에 한다고 했을 때, 이걸 Sync & Blocking 으로 하다보면 
느린 서비스를 만들고 맙니다. 그래서 찾은 방법들을 기술해봅니다.

### 서두 요약 : 100개 이상의 많은 요청을 Async & Non blocking 으로 하는 방법에 관하여

## 1. Parallel Stream
```java
items.parallelStream().forEach(item -> {
  Response result = restTemplate.getForObject(uri, Response.class);
  log.debug("result : {}", result);
}
```
parallelStream 으로 변형하면 core 숫자만큼 Thread 를 만들어서 알아서 실행합니다.
core 숫자는 변경할 수 있지만 아래와 같은 무시무시한 속성 변경을 해야해서 꺼려집니다.
`System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","6");`
오히려 애매한 숫자를 돌리는 경우엔 `parallelStream` 자체로 만드는게 더 일이 많아져 배보다 배꼽이 커지는 상황도 나옵니다.
편하긴 하지만 유연하지 않고, 성능이 더 나빠질 수도 있습니다.

## 2. `@Async` 활용
method 를 Async 하게 실행시키는 방법입니다. 자세한 내용은 [spring.io](https://spring.io/guides/gs/async-method/)에 되어있긴 한데요.
내부에 `RestTemplate` 을 넣고 호출해버리고, [java.concurrent 의 CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) 로 받아올 수 있습니다.

응답 다 기다려서 쓰고싶은 경우, 마지막 `join()` 실행해주면 됩니다. loop 로 도는 경우에도 가능하고 `taskExecutor`를 통해 쓰레드 설정을 할 수 있습니다.
```java
@Bean
public Executor taskExecutor() {
	ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
	executor.setCorePoolSize(2);
	executor.setMaxPoolSize(2);
	executor.setQueueCapacity(500);
	executor.setThreadNamePrefix("GithubLookup-");
	executor.initialize();
	return executor;
}
    
@Async
public CompletableFuture<User> findUser(String user) throws InterruptedException {
	logger.info("Looking up " + user);
	String url = String.format("https://api.github.com/users/%s", user);
	User results = restTemplate.getForObject(url, User.class);
	// Artificial delay of 1s for demonstration purposes
	Thread.sleep(1000L);
	return CompletableFuture.completedFuture(results);
}

@Override
public void run(String... args) throws Exception {
	// Start the clock
	long start = System.currentTimeMillis();

	// Kick of multiple, asynchronous lookups
	CompletableFuture<User> page1 = gitHubLookupService.findUser("PivotalSoftware");
	CompletableFuture<User> page2 = gitHubLookupService.findUser("CloudFoundry");
	CompletableFuture<User> page3 = gitHubLookupService.findUser("Spring-Projects");

	// Wait until they are all done
	CompletableFuture.allOf(page1,page2,page3).join();

	// Print results, including elapsed time
	logger.info("Elapsed time: " + (System.currentTimeMillis() - start));
	logger.info("--> " + page1.get());
	logger.info("--> " + page2.get());
	logger.info("--> " + page3.get());

}
```

## 3. `AsyncRestTemplate` 활용
앞에 두 개는 실제 Http Request 용으로 만든게 아닌데, 얍삽하게 사용을 한 경우고, 남은 두가지는 비동기 요청을 위해 만들어진 녀석들 입니다.
[좀 괜찮은 설명의 링크](http://wonwoo.ml/index.php/post/903)

```java
 @Test
 public void asyncRestTemplateTest() throws InterruptedException, ExecutionException {
     ListenableFuture<ResponseEntity<Map>> entity = asyncRestTemplate.getForEntity("https://httpbin.org/get", Map.class);
     entity.addCallback(result -> {
         System.out.println(result.getStatusCode());
         System.out.println(result.getBody());
     }, ex -> System.out.println(ex.getStackTrace()));
 
     System.out.println("asyncRestTemplateTest");
     TimeUnit.SECONDS.sleep(8);
 }
```
결과는 아래와 같은 순서로 찍힙니다.
```
asyncRestTemplateTest
statusCode
body
```
ListenableFuture 를 통해 javascript ajax 처럼 success 시 ~ callback 등록이 가능합니다. 
단점은 나온지 얼마 되지도 않은 것 같은데, Spring 5 WebClient 나오면서 Deprecated 되었습니다 ㅠㅠㅠ

## 4. `WebClient` 활용
Spring 5.X WebFlux 를 사용한다면 사용 할 수 있습니다.
하지만 대부분 익숙하진 않은 것 같고,, 5.대를 사용하지않으면 사용 할 수가 없는데요. [링크](https://junebuug.github.io/2019-02-11/resttemplate-vs-webclient)
이녀석은 기본이 Async None Blocking 입니다.

저도 심플하게만 써봐서 .. subscribe 에 callback 입력하면 되시겠습니다.
```java
Mono<String> result = WebClient.create()
		.get()
		.uri(uri)
		.retrieve().bodyToMono(String.class);
result.subscribe(log::debug);
```

-----------------------

사실 이렇게 방법이 있어도, 받아주는 쪽 성능에 의지할 수 밖에 없지만, 그쪽 서버를 믿고
비동기로 요청하다보면 1분걸리던게 20초 걸릴때가 오는 것 같습니다 ㅎ
제법 많은 수의 요청을 한다면 비동기 요청을 적극 활용해보는 것도 좋은 경험이 되리라 생각합니다.