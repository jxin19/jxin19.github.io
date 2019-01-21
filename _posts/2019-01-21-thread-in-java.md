---
title: Thread
description: Thread
header: Thread
---

## Process

프로세스는 프로그램이나 어플리케이션에서 볼 수 있는 스스로 포함된 실행환경이다.
프로그램 안에는 여러 프로세스가 포함되어 있다.
Java Runtime 환경에서 돌아가는 단일 프로세스는 각기 다른 클래스와 프로그램 프로세스가 포함되어 있다.

## Thread

Java 어플리케이션에는 최소 1개 이상의 Thread가 있다 - Main Thread

Java Thread는 백그라운드에서 메모리 관리, 시스템 관리, 신호 처리 등을 실행한다.
하지만 어플리케이션 관점에서 보면 Main Thread는 첫번째 Java Thread이고, 다중 Thread를 만들 수 있다.

다중 Thread는 단일 프로그램에서 2개 이상의 Thread가 동시에 실행되는 것을 말한다.
한 컴퓨터 단일 핵심 프로세스는 한번에 단 하나의 Thread만 실행시킬 수 있다. 
그리고 시간을 분할하여 프로세스 시간간 Thread를 공유한다.


## Thread의 혜택 

 - Java Thread는 프로세스에 비해 가볍고 작은 생성 시간과 리소스면 된다.
 - 상위 프로세스 데이터와 코드를 공유한다.
 - Thread 간 Context 전환은 보통 프로세스보다 적은 비용이 든다.
 - Thread 간 통신은 프로세스 통신보다 비교적 쉽다.
 
Java는 2가지 방법으로 Thread를 프로그래밍할 수 있다.

1. java.lang.Runnable 인터페이스 구현
2. java.lang.Thread 클래스 확장


## Thread 예제

#### implementing Runnable 인터페이스

클래스 실행을 가능하게 하기 위해 java.lang.Runnable 인터페이스를 구현할 수 있고, `public void run()` 메소드 구현을 제공한다.
Runnable 클래스의 객체를 전달하여 Thread를 생성한다. 그때 다른 Thread에서 `run()` 메소드를 실행하려면 `start()` 메소드를 호출해야 한다. 

```java
public class HeavyWorkRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("Doing heavy processing - START " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
            //Get database connection, delete unused data from DB
            doDBProcessing();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Doing heavy processing - END " + Thread.currentThread().getName());
    }

    private void doDBProcessing() throws InterruptedException {
        Thread.sleep(5000);
    }

}
```

#### Thread 클래스 확장

We can extend java.lang.Thread class to create our own java thread class and override run() method. 
Then we can create it’s object and call start() method to execute our custom java thread class run method.

java.lang.Thread클래스를 확장하여 Thread 클래스와 override run() 메소드를 만든다.
그리고 이 오브젝트 만들고 start() 메소드를 커스텀 Thread 클래스의 run() 메소드 호출한다.

```java
public class MyThread extends Thread {

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        System.out.println("MyThread - START "+Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
            //Get database connection, delete unused data from DB
            doDBProcessing();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("MyThread - END "+Thread.currentThread().getName());
    }

    private void doDBProcessing() throws InterruptedException {
        Thread.sleep(5000);
    }
    
}
```

```java
public class ThreadRunExample {

    public static void main(String[] args){
        Thread t1 = new Thread(new HeavyWorkRunnable(), "t1");
        Thread t2 = new Thread(new HeavyWorkRunnable(), "t2");
        System.out.println("Starting Runnable threads");
        t1.start();
        t2.start();
        System.out.println("Runnable Threads has been started");
        Thread t3 = new MyThread("t3");
        Thread t4 = new MyThread("t4");
        System.out.println("Starting MyThreads");
        t3.start();
        t4.start();
        System.out.println("MyThreads has been started");
        
    }
}
```

```bash
Starting Runnable threads
Runnable Threads has been started
Doing heavy processing - START t1
Doing heavy processing - START t2
Starting MyThreads
MyThread - START Thread-0
MyThreads has been started
MyThread - START Thread-1
Doing heavy processing - END t2
MyThread - END Thread-1
MyThread - END Thread-0
Doing heavy processing - END t1

```

위 코드를 여러번 실행하면 실행순서를 보장할 수 없다는 것을 알 수 있다.


## Runnable vs Thread

If your class provides more functionality rather than just running as Thread, 
you should implement Runnable interface to provide a way to run it as Thread. 
If your class only goal is to run as Thread, you can extend Thread class.

Implementing Runnable is preferred because java supports implementing multiple interfaces. 
If you extend Thread class, you can’t extend any other classes.

Runnable은 다중 상속이 가능하지만, Thread는 다중상속을 받을 수 없다.
이런 문제 때문에 Runnable을 더 많이 사용한다.


## Thread Pool



> 참고<br/>
> https://www.journaldev.com/1016/java-thread-example#java-thread-benefits
> https://www.journaldev.com/1069/threadpoolexecutor-java-thread-pool-example-executorservice