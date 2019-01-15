---
title: Garbage Collection(가비지 컬렉션)
description: Garbage Collection
header: Garbage Collection(가비지 컬렉션)
---

Garbage Collector가 무엇이며, 어떻게 작동하는지 알아본다.

## Garbage Collector란?

매우 간단히 설명하면 Garbage Collector란 Java에서 사용되지 않는 객체를 자동으로 제거하는 자동 메모리 관리 프로그램이다.
그것은 Heap 메모리에서 처리하고, 사용여부를 식별한다. 사용하지 않는 객체는 제거한다.

```java
for (String name : nameList) {
    String s = name.getName();
}
```

반복할 때마다 만들어진 String 객체는 다음 반복구간에선 사용하지 않는다. 이렇게 참조되지 않는 객체를 Garbage(쓰레기)라 부른다. 
객체를 생성할 때마다 메모리를 사용하게 되고, 반복이 계속될 수록 메모리 소모가 커지게 된다.

<br/>

메모리 관리 개선을 위해 JVM에는 자동 메모리 관리를 수행하는 Garbage Collector가 함께 제공된다.<br/>
(즉, JVM은 메모리를 정리해야 할 때마다 자동으로 메모리를 회수한다.)


## 실제로 Garbage Collection이 어떻게 동작하는가?

Garbage Collector를 오해 하는 이유중 하나는 죽은(더이상 참조하지 않는) 객체를 제거하는 것이다.<br/>
Garbage Collector는 사용중인 객체를 지속적으로 추적하고 다른 것들에게 Garbage라는 표식을 한다.

<br/>

이론적으로 Java Garbage Collector는 매우 단순한 방식으로 동작한다.

1. Garbage Collector는 객체가 더이상 사용되지 않을 때 앞으로 만들어질 객체가 사용할 것을 위해 사용된 객체의 메모리를 확보한다.
2. 명확한 객체 삭제는 없을 것이며 헤드 메모리는 JVM과 함께 남는다.

어떤 객체가 사용중이고 쓰레기인지 확인하기 위해 JVM은 특수 객체인 GC Root를 사용한다.<br/>
프로그램이 Root 객체에 도달할 수 있다면 JVM은 객체를 살아있는 것으로 취급한다.

<br/>

#### Mark and Sweep Algorithm

JVM은 Mark and Sweep Algorithm을 사용하여 사용 중인 개체와 더 이상 사용되지 않는 개체를 확인/표시한다. 이 알고리즘은 2단계로 작동한다.

1. 모든 참조내용을 실행하고 살아 있는(활성) 객체에 표시한다.
2. 살아 있다고 표시되지 않은 모든 객체에 대한 힙(Heap) 메모리를 회수한다.

이 방식은 단순해 보이지만, Garbage Collector는 참조에 따라 객체를 활성 상태로 표시한다는 것을 알아야 한다.<br/>
사용하진 않지만 일부 참조되는 객체는 활성객체로 처리한다.(사용중은 아님)


## Java Garbage Collector 유형

Garbage Collector에 대한 다른 오해는 JVM이 한가지 Garbage Collector만 가지고 있다는 것이다.<br/>
사실은 5개 정도의 Garbage Collector가 있다.(JDK7 기준)<br/>

#### Serial GC

가장 심플하고 유용한 방식이다. 싱글 스레드 환경을 위해 설계되었다.<br/>

**하지만, 사용하지 말 것!**

이슈 중 하나는 Serial GC는 모든 스레드를 멈추게 하는 능력이 있다. 이는 어플리케이션 성능에 심각한 문제를 야기시킨다.

![Serial GC](/img/garbage-collection/serial-gc.jpg)


```bash
-XX:+UseSerialGC
``` 

#### Parallel GC

Java 7, Java 8에서는 기본 GC이다. 멀티 스레드로 GC 프로세스에 대한 힙 탐색을 한다.<br/>
멀티 스레드로 GC를 더욱 빠르게 만들 수 있다. 하지만, GC 작업을 할 때마다 모든 애플리케이션 스레드가 중지된다.<br/>
Parallel GC는 Throughput collector로도 알려져 있다.<br/>

![Parallel GC](/img/garbage-collection/parallel-gc.jpg)


```bash
-XX:+UseParallelGC
``` 

#### CMS GC

Concurrent-Mark-Sweep GC도 가능한 GC 프로세스를 위해 멀티 스레드를 사용하여 헤드를 스캔한다.


#### G1 GC

#### G1 GC and Java 8


> 참고<br/>
> https://www.javadevjournal.com/java/java-garbage-collector/
> http://www.informit.com/articles/article.aspx?p=2496621&seqNum=3
> https://www.techpaste.com/2012/02/java-garbage-collectors-gc/