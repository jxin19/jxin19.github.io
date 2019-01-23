---
title: Lifecycle of Thread
description: Lifecycle of Thread
header: Lifecycle of Thread
---

아래 다이어그램으로 Thread 생명주기(Lifecycle)의 각 상태를 볼 수 있다.
Java에서 Thread를 만들고 시작할 수 있지만, 어떻게 Thread 상태가 Runnable에서 Running to Blocked로 변경되는지는 Thread Scheduler의 OS 구현에 따라 달라지며 이에 대한 제어권은 없다.

![Lifecycle](/img/thread/lifecycle.png)

## New

Thread를 생성할 때 new 생성자를 사용한다.
Thread가 아직 살아있지 않고, Java 프로그래밍 내부 상태이다. 


## Runnable

Thread에서 start() 함수를 호출 할 때, 상태를 Runnable로 바뀐다. 
Thread Scheduler가 실행을 완료하도록 제어권이 주어진다.
Thread 실행여부 또는 실행 전 Thread Pool의 유지여부는 Thread Scheduler의 OS 구현에 따라 달라진다.


## Running

Thread가 실행될 때 상태가 Running으로 바뀐다.
Thread Scheduler는 Thread Pool에서 실행 가능한 Thread 중 하나를 선택하고 실행 상태로 변경한다.
그때 CPU가 해당 Thread를 실행한다.
Thread는 Runnable, Dead, Blocked 상태로 바꿀 수 있다. 
시분할, run() 메소드의 Thread 완료, 일부 리소스를 기다린다.


## Blocked/Waiting

Thread는 다른 Thread가 Thread Join을 사용하여 완료 될 때까지 기다리거나 일부자원을 사용할 수 있을 때까지 기다릴 수 있다.
예를 들어 Producer Consumer 문제, Waiter 알림 구현, IO 리소스의 경우 상태가 대기 중으로 변경된다.
Thread 대기 상태가 끝나면 상태가 Runnable로 변경되고 다시 실행 가능한 Thread Pool로 이동합니다.


## Dead

Thread 실행을 종료하면 상태가 Dead로 변경되고 다시 살아나지 않는다.


> 참고<br/>
> https://www.journaldev.com/1044/thread-life-cycle-in-java-thread-states-in-java