# Future는 무엇인가?

- Java에서 `Future`는 비동기 계산의 아직 계산되지 않은 결과를 표현하는 인터페이스다.

## Future의 주요 메소드

`Future` 인터페이스에는 다음과 같은 주요 메소드들이 있다.

- `get()`: 연산의 결과를 반환한다. 만약 연산이 아직 완료되지 않았다면, 완료될 때까지 기다린다. 이 메소드는 `Future`가 가지는 제네릭 타입의 객체를 반환한다.
- `get(long timeout, TimeUnit unit)`: 지정한 시간 동안만 결과를 기다리고, 그 시간이 지나면 `TimeoutException`을 던진다. 이 메소드는 시간이 지나면 `TimeoutException`을 던지며, 그 전에 작업이 완료되면 그 결과를 반환한다.
- `isDone()`: 연산이 완료되었는지의 여부를 반환한다. 작업이 완료되었다면 `true`, 그렇지 않다면 `false`를 반환한다.
- `cancel(boolean mayInterruptIfRunning)`: 연산을 취소한다. 매개 변수는 작업이 진행 중일 때 중단해도 되는지를 결정한다.
- `isCancelled()`: 작업이 취소되었는지의 여부를 반환한다. 작업이 취소되었다면 `true`, 그렇지 않다면 `false`를 반환한다.

## Future 사용 예시

**예시 1: `Future` 생성 및 사용**

``` java
Callable<Integer> task = () -> {
    Thread.sleep(1000);  // 1초동안 대기
    return 123;  // 결과 반환
};

FutureTask<Integer> future = new FutureTask<>(task);
new Thread(future).start();  // Future 작업 실행

// 다른 작업 ...

Integer result = future.get();  // 결과가 준비될 때까지 Blocking
System.out.println("Result: " + result);
```

---

**예시 2: `ExecutorService`와 `Future`를 이용해 비동기 작업을 실행하기**

```java
// ExecutorService 생성
ExecutorService executor = Executors.newFixedThreadPool(10);

// 비동기 작업 제출
// ExecutorService의 submit 메소드는 Callable이나 Runnable 객체를 인수로 받아서 비동기적으로 실행하며, 결과를 Future 객체로 반환한다.
Future<Integer> future = executor.submit(() -> {
    // 긴 작업 수행
    Thread.sleep(1000);
    return 123;
});

// 다른 작업 수행...

// 결과 가져오기
try {
    Integer result = future.get();  // 결과를 가져온다. 필요한 경우에만 호출한다.
    System.out.println(result);
} catch (InterruptedException | ExecutionException e) {
    // 예외 처리
}

// ExecutorService 종료
executor.shutdown();
```

---

**예시3: `Future`에서 결과 가져오기**

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<Integer> future = executorService.submit(() -> {
    Thread.sleep(2000);  // 임의의 긴 작업 수행
    return 10;
});

try {
    Integer result = future.get();  // 결과가 준비될 때까지 블로킹된다.
    System.out.println(result);     // 10
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
} finally {
    executorService.shutdown();
}
```

---

**예시 4: `Future`에서 타임아웃 설정하여 결과 가져오기**

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<Integer> future = executorService.submit(() -> {
    Thread.sleep(3000);  // 임의의 긴 작업 수행
    return 10;
});

try {
    Integer result = future.get(2, TimeUnit.SECONDS);  // 2초 후에 TimeoutException 발생
    System.out.println(result);
} catch (InterruptedException | ExecutionException | TimeoutException e) {
    e.printStackTrace();
} finally {
    executorService.shutdown();
}
```

---

**예시 5: `Future` 작업 취소하기**

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<Integer> future = executorService.submit(() -> {
    Thread.sleep(3000);  // 임의의 긴 작업 수행
    return 10;
});

future.cancel(true);  // 작업 취소. 작업이 진행 중이라도 중단한다.

try {
    Integer result = future.get();  // CancellationException 발생
    System.out.println(result);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
} finally {
    executorService.shutdown();
}
```

## Future의 장단점과 활용 상황

Java의 `Future` 인터페이스는 비동기 작업의 결과를 추상화하고 관리하는데 매우 유용한 도구다. 하지만 `Future`는 모든 상황에 적합한 도구는 아니다. 그렇기 때문에 `Future`의 장단점과 이를 활용하는 적절한 상황에 대해 이해하는 것이 중요하다.

**장점**

- 비동기 작업의 결과를 쉽게 관리할 수 있다.
- 작업 완료를 기다릴 필요 없이, 결과가 필요할 때 가져올 수 있다.
- `isDone()` 메소드를 통해 작업 완료를 확인할 수 있다.

**단점**

- 작업 완료 알림의 부재: Future는 작업이 완료되었는지 알려주는 메소드(`isDone()`)를 제공하지만, 이 메소드는 polling 방식으로 작동한다. 즉, 계속해서 메소드를 호출하여 작업 완료 여부를 확인해야 한다.
- 연산 결과를 가져올 때 블로킹된다: `get()` 메소드를 호출하면 결과가 준비될 때까지 현재 스레드가 블로킹된다. 이는 `get(long timeout, TimeUnit unit)` 메소드로 일부 해결할 수 있지만, 완벽한 해결책은 아니다.

**활용 상황**

`Future`는 비동기 작업의 결과를 관리하고, 그 결과를 필요로 하는 시점에서 쉽게 가져올 수 있어야 하는 경우에 사용된다. 하지만 작업 완료 알림이 필요하거나, 비동기 작업 간의 종속성을 관리해야 하는 경우에는 Java 8에서 도입된 `CompletableFuture`를 사용하는 것이 더 적절하다. `CompletableFuture`는 `Future`의 단점을 보완하며, 작업 완료 알림, 작업 체이닝, 예외 처리 등의 추가 기능을 제공한다.

Java의 `Future`는 비동기 작업의 결과를 처리하는 강력한 도구다. 하지만 `Future`만으로는 작업이 완료될 때까지 블로킹되거나, 작업의 완료를 주기적으로 폴링해야 하는 등의 단점이 있다. 이러한 문제를 해결하기 위해 Java 8에서는 `CompletableFuture`가 도입되었으며, 이는 `Future`의 기능을 향상시킨 버전이다. `CompletableFuture` 는 `Future`의 단점을 극복하고, 비동기 작업의 완료 알림이나 종속성 관리와 같은 추가적인 기능을 제공한다.

그럼에도 불구하고, `Future`는 여전히 유용하며, 병렬 프로그래밍을 할 때 알아두면 도움이 되는 개념이다.
