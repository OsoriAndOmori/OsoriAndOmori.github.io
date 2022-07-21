## Java Stream 에서의 루프와 Lazy Execution
## 첫번째 깨달음
팀 내 코드리뷰를 하다가 아래와 같은 부분이 추가됨을 보았습니다. 
#### 코드 1
```java
public class Test{
	@Test
	public Optional<Model> fetchModelsById() {
		return fetchModels()
			.stream()
			.filter(distinctByKey(Model::getId))
			.findAny();
	}
	
	private static <T> Predicate<T> distinctByKey(Function<? super T, Object> keyExtractor) {
		Map<Object, Boolean> map = new HashMap<>();
		return t -> map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
	}
}
```
대충 보면 리스트에서 중복된 id의 모델들을 제거하고, 그 중 모델 아무거나 Optional 로 감싸 리턴하는 메서드입니다.
그런데 코드를 보면 이상한 느낌이 처음에 들었습니다.
`distinctByKey 안에 있는 HashMap 을 계속 초기화 하다니 정상동작 하는거 맞아?`
`ConcurrentHashMap` 으로 초기화 해야하는 것이 아닌가? 
혹은 외부에 Map 을 선언해야 하는 것이 아닌가?

예 애석하게도 위 코드는 의도대로 정상동작합니다.

filter() 안에는 true or false 를 리턴하는 Predicate 이 파라미터로 들어가는데요.
**distinctByKey 는 반복되는 로직이아니고, 딱 한번만 실행되어 Predicate 을 생성하고 리턴 된 뒤 더이상 실행 되지 않습니다.**
간단해 보이지만 처음에 이 사실을 받아들이는데 시간이 걸려서 이해하는데 좀 걸렸습니다. 

결국 `t -> map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;` 부분만이 처음에 생성된 map을 참조하여 들고 있고 계속 반복 되는 것이죠.
함수형 프로그래밍 이라는것이 이런걸까요??

-----------------------------------------

## 두번째 깨달음
더불어 java 는 논리 연산에서 `lazy evaluation` 정책을 가지고 있습니다.
```
if (A() || B()){
	System.out.println("TRUE");
}
```
A() 가 true 이면 B()가 true 인지 false 인지 아닌지 관심이 없는 언어죠.
계산 단순화, 빠른실행을 위한 최적화로 불필요한 연산을 최소화 하여 효율적으로 동작하기 위함입니다. 
이는 Stream 에서도 동일하게 적용되어있습니다.

#### 코드 2
```java
final List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
 
Stream stream = list.stream()
        .filter(i -> i<6)
        .filter(i -> i%2==0)
        .map(i -> i*10);
``` 

모든 숫자들이 `filter i->i<6` 거치고 `filter i -> i%2==0` 을 순차적으로 거쳐서 단계별로 넘어가는 것이 아닌, 
가령 `findFirst()`나 `collect()`가 호출 되어서야 lazy evaluation 을 통해 **최종 결과물까지 하나씩 보내는 것** 이죠. 
아래 그림이 잘 표현을 해주고 있습니다.  
 
### Eager evaluation
![9905FD465C4A993D20](https://user-images.githubusercontent.com/22016317/66792825-6b0f9800-ef35-11e9-90f5-15fa3c8ccf0a.gif)

### Lazy evaluation
![99CC3B505C4A994A1C](https://user-images.githubusercontent.com/22016317/66792837-85497600-ef35-11e9-8800-270d7d84d9d7.gif)

이해가 잘 안되는 것 같다는 느낌이 드시면 [lazy evaluation 튜토리얼 코드](https://dzone.com/articles/spring-boot-configure-jetty-server-using-gradle-sp)를 하나씩 돌려보시는 것도 좋습니다.

아래는 java Stream 의 lazy evaluation 을 완벽하게 테스트해볼 수 있는 코드를 만들고 수정해보았습니다.
#### 코드 3
```java
@Slf4j
public class Test {
	@Test
	public void test() throws Exception {
		List<Member> memberList = new ArrayList<>();

		//aa + 숫자로 모델만들어서 넣음
		IntStream.range(13, 17)
				.forEach(i -> {
					memberList.add(new Member("aa" + i, i));
				});

		//unknown + 숫자로 모델 만들어서 넣음
		IntStream.range(10, 20)
				.forEach(i -> {
					memberList.add(new Member("unknown_" + i, i));
				});

		Stream<Member> memberStream = memberList.stream().filter(distinctByKey(m -> m.getAge()));
		System.out.println("NOT EXECUTE YET");
		memberStream.forEach(System.out::println);
	}

	private <T> Predicate<T> distinctByKey(Function<? super T, Object> keyExtractor) {
		System.out.println("Init Map");
		Map<Object, Boolean> map = new HashMap<>();
		return t -> {
			System.out.println("Map Key set : " + map.keySet());
			return map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
		};
	}

	public class Member {
		private String name;
		private long age;

		public Member(String name, long age) {
			super();
			this.name = name;
			this.age = age;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public long getAge() {
			return age;
		}

		public void setAge(long age) {
			this.age = age;
		}

		@Override
		public String toString() {
			return "Member [name=" + name + ", age=" + age + "]";
		}
	}
}
```
output 을 한번 예측해보세요!

### 퀴즈!
코드1 에서 일반 stream 이 아니고 ParallelStream 을 사용한다면 어떻게 해야할까요?
 

### 참고 
- https://dzone.com/articles/spring-boot-configure-jetty-server-using-gradle-sp (이건 한번 따라해보세요.)
- https://www.logicbig.com/tutorials/core-java-tutorial/java-util-stream/lazy-evaluation.html
- https://dororongju.tistory.com/137