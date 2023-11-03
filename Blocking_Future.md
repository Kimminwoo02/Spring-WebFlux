# Blocking/Non-Blocking

# Blocking과 Non-Blocking

1) Blocking

Blocking은 A함수가 B함수를 호출하면 제어권을 A가 호출한 B함수에게 넘겨주게 된다.


1. A함수가 B함수를 호출하면 B에게 제어권을 넘긴다.
2. 제어권을 넘겨받은 B는 함수를 실행한다. A는 B에게 제어권을 넘겨주었기 때문에 함수 실행을 잠깐 멈춘다.
3. B함수는 실행이 끝나면 자신이 호출한 A에게 제어권을 돌려준다.

2) Non-Blocking

A함수가 B함수를 호출해도 제어권을 자신이 그대로 가지고 있는 것이다.


1. A함수가 B함수를 호출하면 B함수는 실행되지만, 제어권은 A함수가 그대로 가지고 있는다.
2. A함수는 계속 제어권을 가지고 있기 때문에 B함수를 호출한 이후에도 자신의 코드를 계속 실행한다. 그래서 별도의 Thread가 필요하다.

### **Blocking의 종류**

- blocking은 Thread가 오랜 시간 일을 하거나 대기하는 경우 발생한다.
- CPU-Bound blocking : 오랜 시간 일을한다.
- IO-Bound- blocking : 오랜 시간 대기한다.

**CPU bound blocking**

thread가 대부분의 시간 CPU를 점유한다. 연산이 많은 경우 추가적인 코어를 투입한다.

**IO bound blocking**

Thread가 대부분의 시간을 대기한다. 파일 읽기/쓰기, network 요청 처리, 요청 전달 등을 한다.

IO bound non-blocking이 가능하다.

**Blocking의 전파**

하나의 함수가 여러 함수를 호출하기도 하고, 함수 호출은 중첩적으로 발생한다. Callee는 caller가 되고 다시 callee를 호출한다. blocking한 함수를 하나라도 호출한다면 caller는 blocking이 된다. 따라서 함수가 non-blocking하려면 모든 함수가 non-blocking이어야 한다. I/O bound blocking 또한 발생하면 안된다.


# Future 인터페이스

### **CompletableFuture**

2014년에 발표된 java8에서 처음 도입되었다. 비동기 프로그래밍을 지원하고 Lamda, Method reference 등 java 8의 새로운 기능을 지원한다.

### Method reference

- :: 연산자를 이용해서 함수에 대한 참조를 간결하게 표현 한 것이다.
    - method reference
    - static method reference
    - instance method reference
    - constructor method reference

```java
@RequiredArgsConstructor
public static class Person{
	@Getter 
	private final String name;

	public Boolean compareTo(Person o){
			return o.name.compareTo(name) > 0;
	}
}

public static void print(String name){
		System.out.println(name);
}

public static void main(String[] args){
	var target = new Person("f");
	consumer<String> staticPrint = MethodReferenceExample::print;

	Stream.of("a","b","g","h")
				.map(Person::new) // constructor reference : 객체 자체에 간단한 참조를 할 수 있다.
				.filter(tagret::compareTo) // method reference : 
				.map(Person::getName) // instance method reference : 특정한 객체에 대한 메소드가 아닌 클래스가 가진 메소드에 대한 참조이다.
				.forEach(staticPrint) // static method reference : Static 에 참조를 할 수 있다.
				
}
```

**CompletableFuture 클래스**

```java
public CompleteFuture<T> implements Future<T>, CompletionStage<T>
```

Future 

- 비동기적인 작업을 수행한다.
- 해당 작업이 완료되면 결과를 반환하는 인터페이스이다.

CompletionStage

- 비동기적인 작업을 수행한다.
- 해당 작업이 완료되면 결과를 처리하거나 다른 CompletionStage를 연결하는 인터페이스이다.

**future 인터페이스**

```java
public interface<V> {
	boolean cancel(boolean mayInterruptRunning);
	boolean isCancelled();
	boolean isDone();
	V get() throws InterruptedException, ExecutionException;
	V get(long timeout, TimeUnit unit)
			Throws InterruptedException, ExecutionException, TimeoutException;
}
```

**ExecutorService**

- 쓰레드 풀을 이용하여 비동기적으로 작업을 실행하고 관리한다.
- 별도의 쓰레드를 생성하고 관리하지 않아도 되므로, 코드를 간결하게 유지 가능하다.
- 쓰레드 풀을 이용하여 자원을 효율적으로 관리할 수 있다.

**ExecutorService 메소드**

- execute: Runnable 인터페이스를 구현한 작업을 쓰레드 풀에서 비동기적으로 실행
- submit : Callable 인터페이스를 구현한 작업을 쓰레드 풀에서 비동기적으로 실행하고, 해당 작업의 결과를 Future<T> 객체로 반환
- shutdown : ExecutorService를 종료. 더이상 task를 받지 않음

**Executors - ExecutorsService 생성**

- newSingleTreadExecutor: 단일 쓰레드로 구성된 스레드 풀을 생성. 한번에 하나의 작업만 실행한다.
- newFixedThreadPool : 고정된 크기의 쓰레드 풀을 생성. 크기는 인자로 주어진 n과 동일
- newCachedThradPool : 사용 가능한 쓰레드가 없다면 새로 생성해서 작업을 처리하고, 있다면 재사용. 쓰레드가 일정 시간 사용되지 않으면 회수
- newScheduledThreadPool : 스케줄링 기능을 갖춘 고정 크기의 쓰레드 풀을 생성. 주기적이거나 지연이 발생하는 작업을 실행
- newWorkStealingPool :work steal 알고리즘을 사용하는 ForkJoinPool을 생성

**FutureHelper**

- getFuture : 새로운 쓰레드를 생성하여 1을 반환
- getFutureCompleteAfter1S : 새로운 쓰레드를 생성하고 1초 대기 후 1을 반환