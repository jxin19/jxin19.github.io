---
title: Streams
description: Streams in Java
header: Streams
tags: [Java, Stream]
---

Java 8에 추가된 가장 대표적인 기능이다. 요소를 순차적으로 처리하는 기능을 가진 클래스이다.

## Stream API

#### Stream Creation
- stream()과 of() 메소드의 도움을 받은 배열 또는 Collection을 만들 수 있다.

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
stream = Stream.of("a", "b", "c");
```

stream() 기본 메소드는 Collection 인터페이스에 추가되었고, Stream<T> 생성을 허용한다.

```java
Stream<String> stream = list.stream();
```

#### Multi-threading with Streams

Stream API는 병렬처리를 하는 parallelStream()를 제공하여 멀티스레딩을 단순화하였다.

아래 코드는 요소의 흐름에 따라 병렬로 처리되는 doWork() 메소드가 실행되는 것을 보여준다.

```java
list.parallelStream().forEach(element -> doWork(element));
```

이제 기본적인 Stream API 몇 가지를 소개한다.


## Stream Operations

Stream으로 수행할 수 있는 유용한 동작들이 많이 있다.Stream의 특징으로 데이터를 변경하지 않고 동작하여 결과를 도출해 낼 수 있다.

```java
long count = list.stream().distinct().count();
```

#### Iterating

Stream API는 for, for-eachl, while 반복문을 대체할 수 있다. 이는 동작 로직에 집중할 수 있게 된다.

이전
```java
for (String string : list) {
    if (string.contains("a")) {
        return true;
    }
}
```

Java 8
```java
boolean isExist = list.stream().anyMatch(element -> element.contains("a"));
```

#### Filtering

filter() 메소드는 요소의 흐름에서 조건에 충족되는 값을 추출할 수 있다.

```java
ArrayList<String> list = new ArrayList<>();
list.add("One");
list.add("OneAndOnly");
list.add("Derek");
list.add("Change");
list.add("factory");
list.add("justBefore");
list.add("Italy");
list.add("Italy");
list.add("Thursday");
list.add("");
list.add("");
```

아래 예제는 List<String> 배열의 Stream에서 'd'가 포함된 요소를 찾아 Stream<String>에 결과를 넣는다.

```java
Stream<String> stream = list.stream().filter(element -> element.contains("d"));
```

#### Mapping

특별한 기능으로 Stream을 변환하고 새로운 Stream으로 만들때는 아래와 같이 map() 방식을 사용할 수 있다.

```java
List<String> uris = new ArrayList<>();
uris.add("C:\\My.txt");
Stream<Path> stream = uris.stream().map(uri -> Paths.get(uri));
```

모든 요소가 순서를 가지고 있는 Stream이고 안쪽 요소에 Stream을 생성하고 싶다면 flatMap()을 사용해야 한다.

```java
List<Detail> details = new ArrayList<>();
details.add(new Detail());
Stream<String> stream = details.stream().flatMap(detail -> detail.getParts().stream());
```

위의 예제는 List<Detail>로 구성된 details를 새로운 결과 Stream에 추가한다. 이후 최초의 Stream<Detail>의 데이터는 손실된다.

#### Matching

Stream API는 몇 가지 시퀀스의 요소를 검증할 수 있는 유용한 기능이 있다.
anyMatch(), allMatch(), noneMatch() 메소드를 boolean 유형으로 결과를 받아 볼 수 있다.

```java
boolean isValid = list.stream().anyMatch(element -> element.contains("h")); // true
boolean isValidOne = list.stream().allMatch(element -> element.contains("h")); // false
boolean isValidTwo = list.stream().noneMatch(element -> element.contains("h")); // false
```

#### Reduction

reduce() 메소드는 Stream의 요소를 단일 값으로 감소시키는 데 사용된다. BinaryOperator를 매개변수로 사용한다.
아래 코드의 결과는 26 (23 + 1 + 1 + 1).

```java
List<Integer> integers = Arrays.asList(1, 1, 1);
Integer reduced = integers.stream().reduce(23, (a, b) -> a + b);
```

#### Collecting

collect() 메소드는 Collection이나 Map으로 변환하고 Stream을 하나의 문자열로 나타낼 때 매우 편리하다.

```java
List<String> resultList = list.stream().map(element -> element.toUpperCase()).collect(Collectors.toList());
```

collect() 메소는 Stream<List>를 List<String> 줄일 때 사용되기도 한다.

> 참고<br/>
> https://www.baeldung.com/java-8-streams
> https://www.baeldung.com/java-8-streams-introduction