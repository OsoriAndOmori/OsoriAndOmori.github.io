# 코딩 컨벤션

코드는 컴퓨터가 읽지만, 사람이 유지보수한다. 내가 작성한 코드를 다시 읽을 때, 팀원이 내 코드를 이해할 때 "의도"가 명확히 전달되어야 한다.

## 아키텍처 & 설계

### 레이어드 아키텍처를 사용할 것

큰 프로젝트일수록 계층을 명확히 한다. 보통 다음처럼:
- **Presentation Layer**: 컨트롤러, 뷰, API 엔드포인트
- **Application/Service Layer**: 비즈니스 로직, 트랜잭션 관리
- **Data Access Layer**: 데이터베이스 쿼리, 엔티티 매핑
- **Domain Layer**: 비즈니스 엔티티, 도메인 규칙

계층이 명확하면 책임이 분리되고, 테스트하기 쉽고, 다른 팀원이 코드를 이해하기 쉽다.

**가장 중요한 규칙: 계층을 뛰어넘지 말 것.**

```kotlin
// ❌ Presentation이 Data Access를 직접 호출 - 계층 위반
@RestController
class UserController(private val userRepository: UserRepository) {
  @GetMapping("/user/{id}")
  fun getUser(@PathVariable id: String): User {
    return userRepository.findById(id)  // Repository 직접 사용
  }
}

// ✅ 계층을 통해서만 접근
@RestController
class UserController(private val userService: UserService) {
  @GetMapping("/user/{id}")
  fun getUser(@PathVariable id: String): User {
    return userService.getUser(id)      // Service를 거쳐서 접근
  }
}
```

Presentation → Service → Repository 순서로만 흐른다. 중간 계층을 건너뛰면 나중에 비즈니스 로직 변경 시 여러 군데를 수정해야 한다.

**In/Out 파라미터의 DTO 설계는 프로젝트 성격에 따라 다르다.** 계층별로 얼마나 엄격하게 분리할지(예: Presentation DTO vs Domain Entity)는 매번 그 프로젝트의 규모와 팀 규약에 따라 결정되므로, 프로젝트를 시작할 때 확인이 필요하다.

### 클래스간 의존성은 한방향으로

의존성은 단방향 흐름을 유지한다. 순환 의존성(A → B → A)이 생기지 않도록 주의한다.

```kotlin
// ❌ 순환 의존성
class UserService(private val orderService: OrderService) { ... }
class OrderService(private val userService: UserService) { ... }

// ✅ 한방향 의존성 + 계층 분리
// UserService는 OrderService를 모르고, OrderService가 필요한 User 정보는 인자로 받음
class UserService { ... }
class OrderService(private val userRepository: UserRepository) { ... }
```

순환 의존성이 보이면, 공통 관심사를 별도 클래스로 추출하거나 의존성 역전(DI)을 고려한다.

### 클래스는 단일책임의 원칙을 지킬 것

한 클래스는 한 가지 이유로만 변경되어야 한다. 클래스가 100줄을 넘으면 "이게 정말 한 책임인가?"를 점검할 타이밍이다.

```kotlin
// ❌ 책임이 많음: 유저 조회 + 이메일 발송 + 데이터 캐싱
class UserManager {
  fun getUser(id: String): User { ... }
  fun sendWelcomeEmail(user: User) { ... }
  fun cacheUser(user: User) { ... }
}

// ✅ 책임을 분리
class UserService { fun getUser(id: String): User { ... } }
class EmailService { fun sendWelcomeEmail(user: User) { ... } }
class UserCache { fun cache(user: User) { ... } }
```

100줄이 넘으면 반사적으로 분리를 고려하자. 클래스가 작을수록 수정할 이유가 적어진다.

---

## 메서드/함수 설계

### 함수명은 어휘처럼 작성하기

함수명을 지을 때 재사용 가능성을 생각하지 않는다. 한 곳에서만 쓰일 거 같아도 `private`으로 빼낸다. 목표는 **"이 로직이 어떤 개념을 표현하는가"를 이름으로 명확히 하는 것**이다.

```kotlin
// ❌ 직접 계산식을 쓰거나 복잡하게 섞어둠
val result = data.filter { it > 10 }.map { it * 2 }.fold(0) { acc, v -> acc + v }

// ✅ 개념 단위로 함수화
private fun filterHighValues(data: List<Int>): List<Int> = data.filter { it > 10 }
private fun doubleValues(data: List<Int>): List<Int> = data.map { it * 2 }
private fun sumValues(data: List<Int>): Int = data.fold(0) { acc, v -> acc + v }

val result = sumValues(doubleValues(filterHighValues(data)))
```

이렇게 하면 나중에 코드를 읽을 때 각 단계가 "무엇을 의도했는가"가 바로 보인다.

#### get/set은 쓰지 말고 더 구체적으로

`get`과 `set`은 너무 추상적이다. 어떤 목적으로 데이터를 얻거나 설정하는지 명확하게 이름 지어야 한다.

```kotlin
// ❌ 뜻이 불명확
fun getUser(id: String): User { ... }
fun setOrderStatus(order: Order, status: String) { ... }
fun getOrders(): List<Order> { ... }

// ✅ 동작이 명확
fun fetchUser(id: String): User { ... }                    // DB에서 조회
fun updateOrderStatus(order: Order, status: String) { ... } // 상태 변경
fun retrieveOrders(): List<Order> { ... }                   // 주문 목록 조회
fun createUser(request: UserCreateRequest): User { ... }    // 사용자 생성
fun deleteOrder(orderId: String) { ... }                    // 주문 삭제
```

각 메서드 이름이 동작을 정확히 표현하면, 호출하는 쪽에서도 "이 함수는 뭘 하는 함수인가"가 바로 이해된다.

### 메서드는 20줄 이상이면 하는 짓이 많지 않은지 생각해볼 것

메서드가 길어질수록 여러 일을 하고 있을 가능성이 높다. 20줄을 넘으면 한 번 멈춰서 생각해본다.

```kotlin
// ❌ 너무 많은 일을 함
fun processOrder(orderId: String): Order {
  val order = fetchOrder(orderId)               // DB 조회
  validateOrder(order)                          // 검증
  val payment = processPayment(order)           // 결제 처리
  val inventory = updateInventory(order)        // 재고 업데이트
  sendConfirmationEmail(order)                  // 이메일 발송
  updateAnalytics(order)                        // 분석 로깅
  return order
}

// ✅ 각 단계를 명확한 메서드로 분리
fun processOrder(orderId: String): Order {
  val order = fetchAndValidate(orderId)
  processPaymentAndInventory(order)
  notifyCustomer(order)
  return order
}

private fun fetchAndValidate(orderId: String): Order { ... }
private fun processPaymentAndInventory(order: Order) { ... }
private fun notifyCustomer(order: Order) { ... }
```

길면 길수록 이해하기 어렵고, 테스트하기 어렵다.

### 메서드 파라미터가 3개를 넘으면 DTO를 만들 생각을 해볼 것

파라미터가 3개를 초과하면, 그것들을 하나의 DTO(Data Transfer Object)로 묶을 수 있는지 생각해본다.

```kotlin
// ❌ 파라미터가 너무 많음
fun createOrder(
  customerId: String,
  itemIds: List<String>,
  quantities: List<Int>,
  shippingAddress: String,
  billingAddress: String,
  paymentMethod: String,
  discountCode: String?
): Order { ... }

// ✅ 관련된 값들을 DTO로 묶음
data class OrderCreateRequest(
  val customerId: String,
  val items: List<OrderItem>,
  val shipping: ShippingInfo,
  val payment: PaymentInfo,
  val discountCode: String?
)

fun createOrder(request: OrderCreateRequest): Order { ... }
```

파라미터를 DTO로 묶으면:
- 메서드 시그니처가 간결해진다
- 관련된 데이터가 하나의 개념으로 명확해진다
- 나중에 파라미터를 추가할 때 DTO만 수정하면 된다 (메서드 시그니처 변경 X)
- 테스트할 때도 요청 객체를 만들기만 하면 되어 간편하다

### void 함수는 없다

리턴값이 없는 함수는 만들지 않는다. 무조건 뭔가 의미 있는 값을 리턴해야 한다.

함수형 프로그래밍에서 말하는 **순수 함수(Pure Function)**는:
- **입력** → **출력** 이 명확해야 한다
- 부수 효과(side effect)를 최소화한다
- void는 "입력에 대한 출력이 없다"는 뜻이므로, 순수 함수의 정의에 맞지 않는다

```kotlin
// ❌ void 함수 = 순수 함수 아님 (부수 효과만 있음)
fun updateUser(user: User) {
  userRepository.save(user)  // 데이터베이스 변경만 함
  // 호출한 쪽은 성공 여부를 알 수 없음
  // 테스트하기도 어려움
}

// ✅ 순수 함수 원칙 (입력 → 출력)
fun updateUser(user: User): User {
  return userRepository.save(user)
}

// 또는 명시적인 성공/실패
fun updateUser(user: User): Result<User> {
  return try {
    Result.success(userRepository.save(user))
  } catch (e: Exception) {
    Result.failure(e)
  }
}
```

리턴값이 있으면:
- **순수 함수에 가까워진다** — 입력과 출력이 명확해서 함수의 의도를 쉽게 파악
- 호출한 쪽에서 성공 여부나 결과를 명확히 처리할 수 있다
- 함수형 프로그래밍 스타일로 값을 연쇄할 수 있다 (`user.updateEmail(...).saveToDB()`)
- 테스트하기 쉽다 (결과를 assertion할 수 있음)
- 부수 효과를 제어할 수 있다 (리턴값을 받지 않으면 호출 자체도 하지 않음)

---

## 변수/객체 설계

### 한 번만 쓰는 값도 변수에 담기

임시값, 중간값, 한 번만 사용할 값도 의미 있는 변수명을 주어 명확히 표현한다. 체이닝된 표현식이나 튜플, 매직 숫자를 그대로 전달하지 않는다.

```kotlin
// ❌ 뜻이 불명확한 튜플, 계산식
return Pair(data.size, data.sum())

// ✅ 의도가 드러나는 변수
val totalCount = data.size
val totalSum = data.sum()
return Pair(totalCount, totalSum)
```

또는:

```kotlin
// ❌ 매직 숫자
delay(5000)

// ✅ 의도가 명확
val retryDelayMillis = 5000
delay(retryDelayMillis)
```

### 변수를 재사용하지 않기

한 번 할당한 변수를 다른 뜻으로 재사용하지 않는다. 같은 이름을 여러 의미로 쓰면 코드 흐름을 따라갈 때 헷갈린다.

```kotlin
// ❌ 변수 재사용
var data = fetchRawData()
println(data)
data = transform(data)  // 같은 이름에 다른 의미
println(data)

// ✅ 각 값에 자기 뜻을 담은 이름
val rawData = fetchRawData()
println(rawData)
val transformedData = transform(rawData)
println(transformedData)
```

처음엔 변수가 많아 보이지만, 각 변수가 뭔지 명확해서 코드 흐름 파악이 쉽다.

### 모든 객체는 immutable로만 만들기

객체를 만들 때 무조건 **불변(immutable)**으로 설계한다. 한 번 생성되면 상태가 바뀌지 않아야 한다.

```kotlin
// ❌ mutable 객체 - 언제든 값이 바뀔 수 있음
data class User(var id: String, var name: String, var email: String) {
  fun updateEmail(newEmail: String) {
    email = newEmail  // 기존 객체 수정
  }
}

// ✅ immutable 객체 - 값 변경 시 새로운 객체 생성
data class User(val id: String, val name: String, val email: String)

val user = User("1", "Alice", "alice@example.com")
val updatedUser = user.copy(email = "newalice@example.com")  // 새로운 객체
```

**Immutable의 장점:**
- **코드 이해가 쉽다** — 한 번 생성된 객체는 절대 변하지 않으므로, 그 객체를 읽으면 항상 같은 상태
- **동시성이 안전하다** — 여러 스레드가 같은 객체를 접근해도 상태 변경 걱정 없음
- **테스트가 쉽다** — 부수 효과가 없어서 입력과 출력만 비교하면 됨
- **디버깅이 쉽다** — 객체가 언제 어디서 변했는지 추적할 필요 없음
- **재사용 가능하다** — 수정을 안 하니까 다른 곳에서 같은 객체를 안심하고 쓸 수 있음

Kotlin의 `data class`와 `copy()`, Java 17+의 `record` 같은 도구들이 immutable 설계를 쉽게 만들어준다.

---

## 코드 스타일

### 줄간격을 엄격하게 유지

- 함수명 다음 줄에 바로 코드 시작
- `return` 다음 줄에서 바로 함수 닫음
- 의미 있는 단위로만 한 줄 정도의 여백을 주어 "호흡"을 준다

```kotlin
private fun processUser(userId: String) {
  val user = fetchUser(userId)
  val preferences = loadPreferences(user)
  
  // 의미있는 구간 - 데이터 검증
  if (!isValid(user)) return null
  
  // 의미있는 구간 - 데이터 변환
  return transform(user, preferences)
}
```

줄간격은 "이 코드의 논리적 구간이 어디까지인가"를 시각적으로 표현하는 도구다.

### 한 줄에 한 로직

한 로직을 여러 줄에 걸쳐 넣지 않는다. 특히 함수 인자가 많을 때 줄을 나누는 것을 피한다.

```kotlin
// ❌ 줄바꿈으로 흩어짐
val myData = fetchMyDataByDateAndType(
  date,
  type
)

// ✅ 한 줄에 끝내기
val myData = fetchMyDataByDateAndType(date, type)
```

함수 인자가 너무 많으면 그게 함수 설계 문제일 가능성이 높다. 줄을 나누는 대신 함수를 재설계하는 게 맞다.

### 체이닝과 모나드로 끝까지 가기

로직을 체이닝할 수 있으면 최대한 활용한다. 모나드 패턴(Kotlin의 `let`, `run`, 또는 WebFlux의 연쇄 오퍼레이터)을 사용해서 응답을 구성하고 바로 반환한다. 중간에 값을 뽑아 변수에 담는 것보다, 마지막까지 monad 안에서 작업하는 편이 깔끔하다.

#### Kotlin 예시

```kotlin
// ❌ 중간에 변수를 빼서 단계별로 처리
val user = getUserById(id)
val preferences = user?.let { loadPreferences(it) }
val finalData = preferences?.let { transform(it) }
return finalData
```

```kotlin
// ✅ 체이닝으로 끝까지 - 뜻이 흐름처럼 읽힘
return getUserById(id)
  ?.let { loadPreferences(it) }
  ?.let { transform(it) }
```

#### WebFlux 예시

```kotlin
// ❌ mono에서 중간에 값을 꺼내서 처리
fun fetchUserData(id: String): Mono<UserData> {
  val userMono = getUserById(id)
  val prefsMono = userMono.flatMap { loadPreferences(it) }
  val finalMono = prefsMono.map { transform(it) }
  return finalMono
}
```

```kotlin
// ✅ 끝까지 mono 체이닝 - 비동기 흐름이 선형으로 보임
fun fetchUserData(id: String): Mono<UserData> {
  return getUserById(id)
    .flatMap { loadPreferences(it) }
    .map { transform(it) }
}
```

모나드를 사용하면 비동기 로직도 동기 코드처럼 읽을 수 있고, 에러 처리도 일관되게 할 수 있다.

---

## 주석은 함수 단위로

코드 중간중간에 설명 주석을 남기지 않는다. 대신 함수나 메서드 위에만 **한 줄 요약**을 작성한다.

이상적으로는 함수명이 의도를 드러내고, 변수명이 역할을 설명해서 주석이 필요 없어야 한다. 하지만 비즈니스 로직이 복잡한 실전에서는 "왜 이렇게 구현해야 하는가"에 대한 **사연**을 남겨야 할 때가 있다.

### 기술 부채, 레거시 제약, API 한계 같은 "기구한 사연"

```kotlin
// Order를 먼저 생성하고 inventory를 나중에 업데이트한다.
// (inventory API가 비동기이고, 시간이 걸려서 order 생성 먼저 해야
// 사용자가 주문 번호를 받을 수 있음 - 3년전 인프라 정책)
private fun createOrderWithInventorySync(order: Order): Order {
  val savedOrder = saveOrder(order)
  inventoryService.updateAsync(order.itemId, order.quantity)
  return savedOrder
}
```

```kotlin
// 특정 고객(ID: 12345)은 우리 시스템과 연동된 외부 결제 게이트웨이 규약상
// 항상 선불 결제만 가능함. 일반 유저와 다른 로직.
// TODO: 2027년 Q2에 게이트웨이 업그레이드 예정 - 그 후 제거 가능
private fun isPrePaymentOnly(customerId: Long): Boolean {
  return customerId == 12345L
}
```

코드만 봐서는 "왜 이렇게 비효율적인가"에 답할 수 없다. 하지만 주석으로 배경을 남기면, 나중에 누가 이 코드를 수정할 때 "아, 이건 단순한 버그가 아니라 이래서 이래야 하는 거구나"하고 이해할 수 있다. 그래야 함부로 "개선"했다가 프로덕션 장애를 내지 않는다.

**함수 단위로 요약만 적되, 필요하면 그 배경을 간결하게 담는다.**

---

이 컨벤션들의 공통점은 **"코드를 읽는 사람(나 포함)이 의도를 빠르게 파악할 수 있도록"** 하는 것이다. 문법적으로 맞는 코드보다 의도가 명확한 코드가 유지보수하기 쉽다.
