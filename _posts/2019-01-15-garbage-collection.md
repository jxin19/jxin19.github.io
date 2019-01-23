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

```bash
-XX:+UseParallelGC
``` 

#### CMS GC

초기 Initial Mark 단계에서는 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝낸다. 
따라서, 멈추는 시간은 매우 짧다. 그리고 Concurrent Mark 단계에서는 방금 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다. 
이 단계의 특징은 다른 스레드가 실행 중인 상태에서 동시에 진행된다는 것이다.

<br/>

그 다음 Remark 단계에서는 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 
마지막으로 Concurrent Sweep 단계에서는 쓰레기를 정리하는 작업을 실행한다. 
이 작업도 다른 스레드가 실행되고 있는 상황에서 진행한다.

<br/>

이러한 단계로 진행되는 GC 방식이기 때문에 stop-the-world 시간이 매우 짧다. 
모든 애플리케이션의 응답 속도가 매우 중요할 때 CMS GC를 사용하며, Low Latency GC라고도 부른다.  

<br/>

하지만 아래와 같은 단점이 있다.

1. 동시성 모드로 동작하기 때문에 다른 방식에 비해 더 많은 메모리와 CPU 점유를 한다.
2. 실행 중인 어플리케이션이 GC 실행 중 힙 상태를 일부 변경한 경우, 업데이트된 참조 정보가 있는지 확인하기 위해 일부 최종단계를 다시 실행한다.
3.  One of the main disadvantages of the CMS GC is encountering [Promotion Failure](https://blogs.oracle.com/poonam/troubleshooting-long-gc-pauses) (Long Pauses) which happens due to the race conditions.

```bash
-XX:+UseConcMarkSweepGC
``` 

![Comparison](/img/garbage-collection/compare-gc.jpg)

#### G1 GC

JDK 7 에서 소개된 새로운 GC가 바로 G1(Garbage First)이다.<br/>
G1 와 다른 방식의 근본적인 차이점 중 하나는 힙을 여러 영역으로 분할하는 것이다.
JVM은 일반적으로 크기가 1에서 32Mb까지 다양한 2천 여 영역을 대상으로 한다.<br/>

G1는 보통 여러 개의 배경 스레드를 사용하여 영역을 스캔하여 대부분이 쓰레기 객체로 이루어진 영역을 선택적으로 골라낸다.

<br/>

다른 방식에 비해 좋은 점은 아래와 같다.

1. 대부분이 쓰레기 객체로 이루어진 영역을 타깃팅하는 것이 다른 방식에 비해 빠르다.
2. 다른 방식에 비해 작게 힙을 사용할 수 있다.
3. 힙을 여러 영역으로 쪼개서 "stop the world"를 피할 수 있다.
4. 대규모 힙 크기와 머신당 여러 JVM인 아키텍쳐 상에서 성능을 향상시킬 수 있다.


```bash
-XX:+UseG1GC
```  

``

#### G1 GC and Java 8

Java 8 에 추가된 기능([String Deduplication](http://openjdk.java.net/jeps/192))은 이렇다.<br/>
문자열은 큰 사이즈의 힙을 점유한다. 새로운 기능은 힙에 중복된 문자열이 들어올 경우 자동으로 동일한 내부 문자열을 제거한다.

```bash
-XX:+UseStringDeduplication
```

자동으로 문자열의 중복 인스턴스를 제거 및 방지하여 힙에 존재하는 데이터를 줄이려고 노력한다.

메모리 관리에서 또 다른 중요한 변화는 PermGen 제거다. JVM이 메모리 관리를 하고, OutOfMemoryError를 처리하기 위함이다.<br/>
[Remove the Permanent Generation](http://openjdk.java.net/jeps/122)

<br/>

> 참고<br/>
> https://www.javadevjournal.com/java/java-garbage-collector/
> http://www.informit.com/articles/article.aspx?p=2496621&seqNum=3
> https://d2.naver.com/helloworld/1329
> https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html