---
title: Asynchronous - Future
description: Future
header: Asynchronous - Future
tags: [Java, Asynchronous, Future]
keywords: 
    - en: Abstract Class
      ko: 추상클래스
      cn: 抽象类[Chōuxiànglèi]
    - en: Asynchronous
      ko: 비동기
      cn: 异步
    - en: Asynchronous Computation
      ko: 비동기식 연산
      cn: 异步计算
    - en: Concurrent Processing
      ko: 동시처리
      cn: 并行处理
    - en: Parallel Processing
      ko: 병렬처리
      cn: 并行处理
    - en: Serial Processing
      ko: 직렬처리
      cn: 串行处理
    - en: Recursion
      ko: 재귀
      cn: 递归[Dìguī]
    - en: Recursive
      ko: 재귀/순환
      cn: 循环[xúnhuán]
---

Future 인터페이스는 Java 5에서 매우 유용하게 쓰였다. 비동고 호출과 동시처리를 할때 사용한다.

## Creating Futures

Future 클래스는 비동기식 연산의 결과를 나타낸다. - 결과는 결국 처리완료 후 Future의 결과가 나타난다.

<br/>

어떻게 Future 인스턴스가 메소드 생성과 반환하는지 알아보자.

<br/>

실행시간이 긴 메소드는 Future 인터페이스와 동시처리를 하기 좋다.
Future의 캡슐화된 작업이 완료될때까지 기다리는 동안에 다른 작업을 실행한다. 

<br/>

Future의 비동기 특성을 활용할 수 있는 예는 아래와 같다.

 - 연산에 집중된 프로세스 (수학적, 과학적 계산)
 - 대용량 데이터 구조 처리 (빅데이터)
 - 메소드 원격호출 (파일 다운로드, HTML 스크랩, 웹 서비스)

#### FutureTask

예를 들어, 매우 간단한 클래스를 만들어 Integer형으로 계산한다.
분명히 "장시간 실행" 메소드는 아니지만, Thread.sleep() 호출하여 마지막 1초를 만들고 완성한다.

```java
public class SquareCalculator {    
     
    private ExecutorService executor = Executors.newSingleThreadExecutor();
     
    public Future<Integer> calculate(Integer input) {        
        return executor.submit(() -> {
            Thread.sleep(1000);
            return input * input;
        });
    }
}
```

실제 수행되는 코드는 람다 표현식을 제공하는 call() 메소드를 포함하고 있다. 
보다시피 앞서 언급한 sleep() 호출 이외에는 특별한 것이 없다.

<br/>

직접 Callable과 ExecuterService에 관심을 가질 때 더욱 흠미로워진다.

<br/>

Callable 인스턴스를 생성하는 것은 어디로 데려가는 것은 아니다. 
여전히 인스턴스를 Thread에서 작업을 시작하고 값이 있는 Future 객체를 돌려주는 실행자에 전달해야 한다.
ExecutorService가 있는 곳이다.

<br/>

ExecutorService 인스턴스를 얻을 수 있는 방법은 몇가지 있다.
대부분 유틸리티 클래스는 Executors의 static factory 메소드를 제공한다.
이 예제에서는 기본으로 newSingleThreadExecutor()를 사용한다. 
이것은 한번에 하나의 Thread를 처리할 수 있는 ExecutorService 를 제공한다.

<br/>

ExecutorService 객체를 갖게 되면 Callable의 인자로서 전달하는 submit()을 호출할 필요가 있다.
submit()은 Task 시작하고 Future 인터페이스를 구현하는 FutureTask 객체를 반환할 것이다.


## Consuming Futures

위에서 Future 인스턴스를 어떻게 만드는지 알아보았다.
이번 섹션은 Future API의 일부인 모든 메소드를 탐색하여 어떻게 인스턴스와 동작하는지 알아보자.

#### isDone(), get() 을 사용하여 결과받기

calculate() 호출이 필요하고, Future를 사용하여 결과로 Integer를 반환한다.
Future API의 두 가지 방법이 작업에 도움이 될 것이다.

<br/>

Future.isDone()은 executor가 작업 처리를 완료했는지 알려준다.
만약 작업이 종료되면, true나 false를 반환한다.

<br/>

이 메소드는 계산한 Future.get() 실제 결과를 반환한다.
참고로 작업이 끝날때까지 메소드의 실행을 막는다.
하지만 일례로 isDone()을 호출하여 작이 완료되었는지 먼저 확인하므로 문제가 되지 않는다.

<br/>

위 두 메소드는 주요 작업의 종료를 기다렸다 끝날 때까지 코드의 실행을 기다린다.  

```java
Future<Integer> future = new SquareCalculator().calculate(10);

while(!future.isDone()) {
    System.out.println("Calculating...");
    Thread.sleep(300);
}

Integer result = future.get();
```

예를 들어, 간단한 메시지를 출력하여 사용자가 프로그램 계산을 수행하고 있음을 알 수 있게 한다. 

<br/>

메소드 get()은 작업이 종료될 때까지 실행을 막는다.
하지만 걱정할 필요 없다. 이 예제는 get() 메소드를 호출하여 작업이 완료되었는지 확인한 후 실행한다.
이 시나리오는 future.get()이 항상 즉시 결과를 반환한다.

<br/>

get()은 시간초과와 TimeUnit을 인수로 사용하는 오버라이드 버전이다.  

```java
Integer result = future.get(500, TimeUnit.MILLISECONDS);
```

get(long, TimeUnit)과 get()은 차이는 만약 시간초과(Timeout) 전에 작업이 반환되지 않을 경우 TimeoutException 예외처리를 한다.

#### Future에서 cancel()로 취소처리

작업을 트리거 처리했다고 보자, 어떤 이유에서든 결과를 보장할 수 없다.
Future.cancel(boolean)을 사용하여 실행자에게 작업을 중지하고 기본 Thread를 중단하라고 말할 수 있다.

```java
Future<Integer> future = new SquareCalculator().calculate(4);

boolean canceled = future.cancel(true);
```

위 코드는 작업을 완료할 수 없는 Future 인스턴스이다.
사실 인스턴스에서 get()을 호출시도하면 cancel()을 호출한 후 CancellationException이 출력된다.
Future.isCancelled()는 Future가 이미 취소되었는지 알려준다. 이것은 CancellationException를 피하는 매우 유용한 방법이다.

<br/>

cancel()을 호출이 실패할 수 있다. 예를 들어 반환값이 false일 경우이다.
cancel()은 boolean 값 인수를 가진다는 것을 참고하자.


## More Multithreading with Thread Pools

ExecutorService는 Executors.newSingleThreadExecutor와 함께 존재하기 때문에 단일(싱글) Thread로 처리한다.
단일 Tread를 강조하기 위해 다음 두 연산을 동시에 트리거해 보자.

```java
SquareCalculator squareCalculator = new SquareCalculator();

Future<Integer> future1 = squareCalculator.calculate(10);
Future<Integer> future2 = squareCalculator.calculate(100);

while (!(future1.isDone() && future2.isDone())) {
    System.out.println(
      String.format(
        "future1 is %s and future2 is %s", 
        future1.isDone() ? "done" : "not done", 
        future2.isDone() ? "done" : "not done"
      )
    );
    Thread.sleep(300);
}

Integer result1 = future1.get();
Integer result2 = future2.get();

System.out.println(result1 + " and " + result2);

squareCalculator.shutdown();
```

코드의 출력결과를 분석해 보자.

```bash
calculating square for: 10
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
calculating square for: 100
future1 is done and future2 is not done
future1 is done and future2 is not done
future1 is done and future2 is not done
100 and 10000
```

이것은 명백히 병렬처리가 아니다. 
첫번째 작업이 완료되고 두번째 작업이 시작된다는 것에 주목하자. 전체 프로세스 수행시간이 2초가되었는지 보자.

<br/>

다중 Thread로 프로그램을 만들려면 ExecutorService의 다른 형태를 사용해야 한다.
Executors.newFixedThreadPool() Factory 메소드를 제공한 Thread Pool을 사용할 경우, 아래 예제로 어떻게 동작하는지 보자.

```java
public class SquareCalculator {
  
    private ExecutorService executor = Executors.newFixedThreadPool(2);
     
    //...
}
```

SquareCalculator 클래스를 간단하게 수정하였다. 이제 2개의 Thread를 동시에 사용할 수 있는 executor가 생겼다.
나머지 코드를 입력하여 결과를 출력해보면 아래와 같다.

```bash
calculating square for: 10
calculating square for: 100
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
100 and 10000
```

이 방법이 더 낫다고 본다. 어떻게 2개의 작업의 시작과 종료를 동시에 수행하는지, 전체 프로세스 수행시간이 1초인지 알아보자.
다른 Factory Method는 Thread Pool을 만드는데 사용가능하다. 아래와 같은 것들이 있다.
 - 이전 Thread를 재사용 가능한 Executors.newCachedThreadPool()
 - 특정 지연 후 명령을 예약하는 Executors.newScheduledThreadPool()


## Overview of ForkJoinTask

ForkJoinTask는 추상클래스이다. Future를 구현하고, ForkJoinPool의 사실상 Thread의 대량 작업 실행이 가능하다.

<br/>

이번 섹션은 ForkJoinPool의 주요 특징을 빠르게 보완할 수 있다.
포괄적인 내용은 [Fork/Join Framework in Java](https://www.baeldung.com/java-fork-join)를 참고하자.

<br/>

ForkJoinTask의 주요 특징은 일반적으로 주요 작업을 완료하는데 필요한 작업의 일부를 새 하위 작업으로 생성한다.
fork()를 호출하고 join()를 사용하여 모든 결과를 수집하여 클래스 이름이 된다.

<br/>

ForkJoinTask을 구현하는 두 추상 클래스가 있다.
 - 완료시 값을 리턴하는 RecursiveTask
 - 아무것도 반환하지 않는 RecursiveAction 

<br/>

이름에서 알 수 있듯이 이러한 클래스는 예를 들어 파일 시스템 탐색이나 복잡한 수학 계산과 같은 재귀 작업에 사용됩니다.

<br/>

이전 예제를 확장하여 만든 클래스는 아래와 같다. Integer의 모든 요소에 대한 팩토리얼(factorial)로 계산하는 클래스를 만들어 보았다.
예를 들어 숫자 4를 인수에 넣고 계산하면 4² + 3² + 2² + 1²의 합으로부터 30을 얻는다.

<br/>

우선 RecursiveTask의 구체적인 구현체를 생성하고 compute() 메소드를 구현해야한다.

```java
public class FactorialSquareCalculator extends RecursiveTask<Integer> {

    private Integer n;

    public FactorialSquareCalculator(Integer n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {
        if (n <= 1) {
            return n;
        }

        FactorialSquareCalculator calculator = new FactorialSquareCalculator(n - 1);

        calculator.fork();

        return n * n + calculator.join();
    }
}
```

compute() 메소드 안에 FactorialSquareCalculator의 새 인스턴스를 생성하여 재귀처리하는 방법을 참고한다.
논블로킹 메소드 fork()를 호출함으로써 ForkJoinPool에게 하위 작업을 실행하도록 요청한다.

<br/>

join() 메소드는 숫자의 제곱을 더하는 계산결과를 반환할 수 있다.

<br/>

이제 실행과 Thread 관리를 처리하기 위해 ForkJoinPool을 만들어야 한다.

```java
ForkJoinPool forkJoinPool = new ForkJoinPool();

FactorialSquareCalculator calculator = new FactorialSquareCalculator(10);

forkJoinPool.execute(calculator);
```


> 참고<br/>
> https://www.baeldung.com/java-future
