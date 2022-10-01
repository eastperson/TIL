# Future

- 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.

https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<String> helloFuture = executorService.submit(() -> {
    Thread.sleep(2000L);
    return "Callable";
});

System.out.println("Hello");

String result = helloFuture.get();
System.out.println(result);

executorService.shutdown();
```

- 블로킹 콜이다.
- 타임아웃을 설정할 수 있다.

작업 상태 확인하기 isDone()
  - 완료 했으면 true 아니면 false를 리턴한다.

작업 취소하기 cancel()
  - 취소 했으면 true 못했으면 false를 리턴한다.
  - parameter로 true를 전달하면 현재 진행중인 쓰레드를 interrupt하고 그러지 않으면 현재
진행중인 작업이 끝날때까지 기다린다.

여러 작업 동시에 실행하기 invokeAll()
  - 동시에 실행한 작업 중에 제일 오래 걸리는 작업 만큼 시간이 걸린다.

여러 작업 중에 하나라도 먼저 응답이 오면 끝내기 invokeAny()
  - 동시에 실행한 작업 중에 제일 짧게 걸리는 작업 만큼 시간이 걸린다.
  - 블록킹 콜이다.

# Future의 문제점

- Future는 외부에서 완료시킬 수 없다. (취소하거나 get()에 타임아웃 설정할 수는 있다.)
- 블로킹 코드(get())를 사용하지 않고서는 작업이 끝났을 때 콜백을 실행할 수 없다.
- 여러 Future를 조합할 수 없다.
  - 예) Event 정보 가져온 다음 Event에 참석하는 회원 목록 가져오기
- 예외 처리용 API를 제공하지 않는다.

```kotlin
val executorService = Executors.newFixedThreadPool(4)
val future = executorService.submit(() -> "hello");
```

# CompletableFuture

**장점**
- 스레드의 선언 없이 비동기 연산 작업을 구현할 수 있고 병렬 프로그래밍이 가능
- 파이프라인 형태로 작업을 연결할 수 있어서 비동기 작업의 순서를 정의하고 관리할 수 있다.

[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
- Implements Future
- Implements [CompletionStage](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)

## 비동기로 작업 실행하기

### 리턴값이 없는 경우: runAsync()
- Runnable 구현체를 이용해서 비동기 연산 작업을 하기 위한 새로운 CompletableFuture 객체를 리턴한다.
- Runnable의 run 메서드는 void 타입이기 때문에 값을 외부에 리턴할 수 없다. (스레드 실행 결과를 확인할 수 없다)

```java
public static CompletableFuture<Void> runAsync(Runnable runnable) {
	return asyncRunStage(ASYNC_POOL, runnable);
}
```

```kotlin
@Test
fun `runAsync`() {
	val completableFuture = CompletableFuture.runAsync {
		// [ForkJoinPool.commonPool-worker-19] : Hello
		println("[${Thread.currentThread().name}] : Hello")
	}
}
```

### 리턴값이 있는 경우: supplyAsync()
- Supplier 구현체를 이용해서 비동기 연산 작업을 하기 위한 새로운 CompletableFuture 객체를 리턴한다.
  - Supplier: 파라미터 없이 값만 있는 경우 사용
- Supplier는 리턴 값이 있기 때문에 받아서 결과를 확인할 수 있다.

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
	return asyncSupplyStage(ASYNC_POOL, supplier);
}
```

```kotlin
@Test
fun `supplySync`() {
	val completableFuture = CompletableFuture.supplyAsync {
		println("[${Thread.currentThread().name}] : Hello")
		"Hello"
	}
    assertThat(completableFuture.get()).isEqualTo("Hello")
}
```

- 원하는 Executor(쓰레드풀)를 사용해서 실행할 수도 있다. (기본은 ForkJoinPool.commonPool())

## 콜백 제공하기

### thenApply(Function): 리턴값을 받아서 다른 값으로 바꾸는 콜백
- 현재 단계가 성공적으로 종료되었을 경우, 메서드의 파라미터로 전달된 Function 구현체를 실행하기 위한 CompletionStage 객체를 리턴한다.
  - Function : 전달할 파라미터를 다른 값으로 변환해서 리턴할 때 사용
- 리턴 값을 받아서 다른 값으로 바꾸는 콜백
- ex) API 요청 후 리턴 값을 DTO로 변환

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
	return uniApplyStage(null, fn);
}
```

```kotlin
@Test
fun `thenApply 리턴 값을 받아 다른 값으로 바꾸는 콜백`() {
	val completableFuture = CompletableFuture.supplyAsync {
		println("[${Thread.currentThread().name}] : Hello")
		"Hello"
    }.thenApply { s ->
		println("[${Thread.currentThread().name}] : $s")
        s.uppercase()
    }

    assertThat(completableFuture.get()).isEqualTo("HELLO")
}
```

### thenAccept(Consumer): 리턴값을 또 다른 작업을 처리하는 콜백 (리턴없이)

- 현재 단계가 성공적으로 종료되었을 경우, 메서드의 파라미터로 전달된 Consumer 구현체를 실행하기 위한 CompletionStage 객체를 리턴한다.
  - Consumer : 파라미터를 전달해서 처리한 후 결과를 리턴받을 필요가 없을 때 사용
- 리턴 값을 또 다른 작업으로 처리하는 콜백
- ex) API 요청 작업 후 리턴 값을 로깅

```java
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
	return uniAcceptStage(null, action);
}
```

```kotlin
@Test
fun `thenAccept 리턴 값을 받아 다른 작업으로 처리하는 콜백`() {
	val completableFuture = CompletableFuture.supplyAsync {
		println("[${Thread.currentThread().name}] : Hello")
        "Hello"
	}.thenAccept { s ->
		println("[${Thread.currentThread().name}] : $s")
	}
	completableFuture.get()
}
```



### thenRun(Runnable): 리턴값 받지 다른 작업을 처리하는 콜백

- 현재 단계가 성공적으로 종료되었을 경우, 메서드의 파라미터로 전달된 Runnable 구현체를 실행하기 위한 CopletionState 객체를 리턴한다.
- 결과 값 없이 다음 작업을 실행하는 콜백

```java
public CompletableFuture<Void> thenRun(Runnable action) {
	return uniRunStage(null, action);
}
```

```kotlin
@Test
fun `thenRun 다음 작업 실행하는 콜백`() {
	val completableFuture = CompletableFuture.runAsync {
		println("[${Thread.currentThread().name}] : Hello")
    }.thenRun {
        println("[${Thread.currentThread().name}] : World")
    }
}
```

콜백 자체를 또 다른 쓰레드에서 실행할 수 있다.

## 조합하기

### thenCompose(): 두 작업이 서로 이어서 실행하도록 조합

```kotlin
@Test
fun `thenCompose 두 작업을 이어서 실행`() {
	val helloWorld = CompletableFuture.supplyAsync {
			"Hello"
	}.thenCompose { s -> CompletableFuture.supplyAsync { "$s World" } }

	assertThat(helloWorld.get()).isEqualTo("Hello World")
}
```

### thenCombine(): 두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행

### allOf(): 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행

### anyOf(): 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행

## 예외처리

### exeptionally(Function)

```kotlin
@Test
fun `exceptionally 예외 처리`() {
	val throwError = true
	val hello = CompletableFuture.supplyAsync {
      if (throwError) {
        throw IllegalArgumentException()
      }
      println("[${Thread.currentThread().name}] : Hello")
          "Hello"
    }.exceptionally { error ->
		println(error)
        "Error"
    }
    assertThat(hello.get()).isEqualTo("Error")
}

```

### handle(BiFunction):

참고
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html
- https://github.com/hyenny/posting-review/blob/314278223f9fb850e658c2e73f638c9412ac9438/hyeinkim/2022-09-24-completablefuture.md
