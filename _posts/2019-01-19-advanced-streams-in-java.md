---
title: Streams - Advanced
description: Streams - Advanced
header: Streams - Advanced
---

## Stream Creation

Stream을 만들 때 인스턴스는 소스가 되는 데이터를 편집할 수 없다. 소스 데이터로 여러 인스턴스를 만들 수 있다.  

#### Empty Stream

빈 Stream을 생성할 때 empty() 메소드는 사용한다. 

```java
Stream<String> streamEmpty = Stream.empty();
```

요소가 없는 스트림에 대해 null을 반환하지 않기 위해 생성 시 empty() 메서드를 사용하는 경우가 많다.

```java
public Stream<String> streamOf(List<String> list) {
    return list == null || list.isEmpty() ? Stream.empty() : list.stream();
}
```

#### Stream of Collection

Collection, List, Set 유형에도 Stream을 사용한다.

```java
Collection<String> collection = Arrays.asList("a", "b", "c");
Stream<String> streamOfCollection = collection.stream();
```

#### Stream of Array

Array를 Stream에 넣을 수 있다. 

```java
Stream<String> streamOfArray = Stream.of("a", "b", "c");
```

이미 존재하는 Array에 내용을 추가할 수 있다.

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> streamOfArrayFull = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);
```

#### Stream.builder()

builder()는 Stream에 원하는 값을 추가하여 build하면 Stream으로 반환한다.

```java
Stream<String> streamBuilder = Stream.<String>builder().add("a").add("b").add("c").build();
```

#### Stream.generate()

Stream의 결과를 무한으로 받을 수 있다. 개발자가 크기를 지정해야 한다. 그렇지 않으면 메모리가 한계에 도달할 수 있다.

```java
Stream<String> streamGenerated = Stream.generate(() -> "element").limit(10);
```

위 코드는 10개의 'element'가 들어간 Stream이 생성되었다. 

#### Stream.iterate()

무한한 Stream을 만들 수 있는 다른 방법은 iterate() 메소드이다.

```java
Stream<Integer> streamIterated = Stream.iterate(40, n -> n + 2).limit(20); // 40, 42, 44, 46 ... 
```

첫번째 반복의 결과는 iterate() 메소드의 첫번째 파라미터 값이다. 매 반복마다 이전 반복의 결과값으로 계산된다.

#### Stream of Primitives

Java 8은 3가지의 기본유형(int, long, double)으로 Stream을 만들 수 있다.
Stream<T>는 제네릭 인터페이스이고, 일반 인터페이스와 함께 기본유형을 사용할 수 없어 IntStream, LongStream, DoubleStream의 새로운 특수 인터페이스가 생겼다.

새로운 인터페이스를 사용하면 불필요한 [Auto-boxing](http://minjoon.com/autoboxing-and-unboxing)을 하지 않아 성능을 높일 수 있다.


```java
IntStream intStream = IntStream.range(1, 3);
LongStream longStream = LongStream.rangeClosed(1, 3);
```

range(int startInclusive, int endExclusive) 메소드는 startInclusive 부터 endExclusive 전까지 1씩 증가한다.

rangeClosed(int startInclusive, int endInclusive) 메소드는 range() 메소드와 동일하고 endInclusive 까지 포함하여 결과를 낸다.
 
아래와 같이 Random 클래스도 범위를 지정하여 DoubleStream을 생성할 수 있다. 
 
```java
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);
```

#### Stream of String

String을 활용하여 Stream을 만들 수도 있다.

String 클래스의 chars() 메소드로 Stream을 만드는데 사용할 수 있다. JDK에는 CharStream 이 없기 때문에 대신 chars()를 이용한다.

```java
IntStream streamOfChars = "abc".chars();
```

아래 예제는 정규표현식에 따라 콤마로 구분하여 String을 Stream으로 만드는 코드이다.

```java
Stream<String> streamOfString = Pattern.compile(", ").splitAsStream("a, b, c");
```

#### Stream of File

NIO 클래스 Files을 활용하여 텍스트파일의 Stream<String>로 만들 수 있다.
모든 열이 Stream의 요소가 된다.

```java
Path path = Paths.get("C:\\file.txt");
Stream<String> streamOfStrings = Files.lines(path);
Stream<String> streamWithCharset = Files.lines(path, Charset.forName("UTF-8"));
```


## Referencing a Stream 

It is possible to instantiate a stream and to have an accessible reference to it as long as only intermediate operations were called. Executing a terminal operation makes a stream inaccessible.

To demonstrate this we will forget for a while that the best practice is to chain sequence of operation. Besides its unnecessary verbosity, technically the following code is valid:

```java
Stream<String> stream = Stream.of("a", "b", "c").filter(element -> element.contains("b"));
Optional<String> anyElement = stream.findAny();
```

그러나 터미널 작동을 호출한 후 동일한 참조를 재사용하려고 하면 IllegalStateException 처리를 한다.


```java
Optional<String> firstElement = stream.findFirst();
```

Java 8 Stream은 재사용할 수 없다. 이것은 매우 중요한 내용이다.

이러한 종류는 Stream이 요소를 저장하진 않지만 기능적으로 한정된 일련의 작업을 할 수 있는 기능을 제공하도록 설계되었다. 때문에 매우 논리적이라고 할 수 있다.

위에서 재사용할 수 없던 문제는 아래와 같이 해결 할 수 있다.

```java
List<String> elements = Stream.of("a", "b", "c").filter(element -> element.contains("b")).collect(Collectors.toList());
Optional<String> anyElement = elements.stream().findAny();
Optional<String> firstElement = elements.stream().findFirst();
```


## Lazy Invocation

중간과정이 지연처리된다. 이 말은 터미널 작업실행에 필요할 때만 호출된다는 것이다.

이를 확인하기 위해 호출될 때마다 내부 카운터가 증가하는 wasCalled() 메소드를 만들어 봤다.

```java
private long counter;
  
private void wasCalled() {
    counter++;
}
```

filter()를 실행 할때 wasCalled() 메소드가 실행되도록 해보자.

```java
List<String> list = Arrays.asList(“abc1”, “abc2”, “abc3”);
counter = 0;
Stream<String> stream = list.stream().filter(element -> {
    wasCalled();
    return element.contains("2");
});
```

List<String> 에 세 개의 요소가 담겨 있고 filter() 메소드는 3번을 호출하게 된다. counter 변수값은 3이 될 것이라 생각할 수 있다.
하지만 위 코드의 결과는 여전히 0이다. filter() 메소드는 한번도 wasCalled() 메소드를 호출하지 않았다. 그 이유는 마지막 터미널 동작이 누락되었기 때문이다.

map() 을 추가하여 위 코드를 아래와 같이 수정해야 원하는 결과를 볼 수 있다. 정확한 결과를 보기 위해 로그를 남겨 보자.

```java
Optional<String> stream = list.stream().filter(element -> {
    log.info("filter() was called");
    return element.contains("2");
}).map(element -> {
    log.info("map() was called");
    return element.toUpperCase();
}).findFirst();
```

위 코드의 로그를 보면 filter() 메소드는 두번 호출 되었고, map() 메소드는 한번 호출되었다. 파이프라인은 수직적으로 실행되기 때문이다.

list의 첫번째 요소는 filter() 메소드에 조건에 부합하지 않아 다음 요소로 넘어갔고, 세번째 요소는 호출되지 않고 map() 메소드로 넘어갔다.

findFirst()는 단 한가지 요소만 만족하여 실행된다. 따라서 지연처리는 filter(), map() 호출을 모두 피할 수 있다.

## Order of Execution

성능면에서 봤을 때, Stream 파이프라인내의 체인 동작의 가장 중요한 면중 하나는 올바른 순서이다.

```java
long size = list.stream().map(element -> {
    wasCalled();
    return element.substring(0, 3);
}).skip(2).count();
```

위 코드를 실행하면 값이 3 증가하게 된다. 이 말은 map() 메소드가 세번 호출된다는 것이다.
하지만 값의 크기는 1이다. Stream의 결과는 단 하나의 요소이고, 세번의 호출중 두번은 의미없이 map() 메소드를 실행한 것이다.

만약 skip() 메소드와 map() 메소드의 위치를 바꾸면 counter는 1 증가하게 된다. 

```java
long size = list.stream().skip(2).map(element -> {
    wasCalled();
    return element.substring(0, 3);
}).count();
```

이는 다음과 같은 규칙을 제시한다. 각 요소에 적용되는 작업 전에 Stream의 크기를 줄이는 중간 작업을 배치해야 한다.
따라서 skip(), filter(), distinct() 메소드를 Stream 파이프라인 상단에 배치해야 한다.


## Stream Reduction

Stream API는 count(), max(), min(), sum()과 같은 기본적인 유형이 있다. 
하지만, 이런 유형으로 동작하는 것은 사전에 정의된 구현에 따라 작동한다. 
개발자는 Stream의 감소 메커니즘을 사용자 정의해야 한다면? 이를 가능케 할 수 있는 reduce(), collect() 메소드가 있다.

#### The reduce() Method

시그니처와 리턴 유형에 따라 3가지 방법이 있다.

Identity - Accumulator(누적기)의 초기값 또는 Stream이 비어 있어 누적 될 것이 없는 경우.

Accumulator - 요소의 집합 논리르 지정하는 함수. Accumulator가 감소하는 단계에서 새로운 값을 만든다. 
새 값의 양은 Stream의 크기와 동일하며 마지막 값만 쓸모 있다. 이것은 좋은 성능을 보여주지 못한다.

```java
OptionalInt reduced = IntStream.range(1, 4).reduce((a, b) -> a + b);
```

reduced = 6 (1 + 2 + 3)

```java
int reducedTwoParams = IntStream.range(1, 4).reduce(10, (a, b) -> a + b);
```

reducedTwoParams = 16 (10 + 1 + 2 + 3)

```java
int reducedParams = Stream.of(1, 2, 3)
  .reduce(10, (a, b) -> a + b, (a, b) -> {
     log.info("combiner was called");
     return a + b;
  });
```

위 코드의 결과도 reducedTwoParams과 동일하게 16이 나온다.

```java
int reducedParallel = Arrays.asList(1, 2, 3).parallelStream()
    .reduce(10, (a, b) -> a + b, (a, b) -> {
       log.info("combiner was called");
       return a + b;
    });
```

이번 결과는 36이다. 세번에 걸쳐 병렬로 계산된 결과 이다. (10 + 1 = 11; 10 + 2 = 12; 10 + 3 = 13;)


#### The collect() Method

```java
List<Product> productList = Arrays.asList(new Product(23, "potatoes"),
  new Product(14, "orange"), new Product(13, "lemon"),
  new Product(23, "bread"), new Product(13, "sugar"));
```

Stream을 Collection, List or Set으로 변환한다.

```java
List<String> collectorCollection = productList.stream().map(Product::getName).collect(Collectors.toList());
```

아래 코드는 String 형으로 줄인다.

```java
String listToString = productList.stream().map(Product::getName).collect(Collectors.joining(", ", "[", "]"));
```

많은 요소의 평균값을 구할때는 아래와 같이 구현할 수 있다. 

```java
double averagePrice = productList.stream().collect(Collectors.averagingInt(Product::getPrice));
```

많은 요소의 합계를 구할때는 아래와 같다.

```java
int summingPrice = productList.stream().collect(Collectors.summingInt(Product::getPrice));
```

## Parallel Streams

Java 8 이전의 병렬화는 복잡했다. ExecutorService와 ForkJoin은 개발을 보다 간단하게 만들어 줬다.
하지만 여전히 명확한 실행부를 어떻게 만들지 어떻게 실행할지 유념해야 한다.

Stream API를 사용하면 병렬 모드에서 동작하는 병렬 Stream을 만들 수 있다. Stream의 소스가 Collect나 Array일 경우, parallelStream() 메소드를 사용할 수있다. 

```java
Stream<Product> streamOfCollection = productList.parallelStream();
boolean isParallel = streamOfCollection.isParallel();
boolean bigPrice = streamOfCollection
  .map(product -> product.getPrice() * 12)
  .anyMatch(price -> price > 200);
```

Stream의 소스가 Collect나 Array이 다를경우에는 parallel() 메소드를 사용해야 한다.

```java
IntStream intStreamParallel = IntStream.range(1, 150).parallel();
boolean isParallel = intStreamParallel.isParallel();
```

sequential() 메소드는 병렬 Stream을 다시 순차모드로 변경할 수 있다.

```java
IntStream intStreamSequential = intStreamParallel.sequential();
boolean isParallel = intStreamSequential.isParallel();
```


> 참고<br/>
> https://www.baeldung.com/java-8-streams