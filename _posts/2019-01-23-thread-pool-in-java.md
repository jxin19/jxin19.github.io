---
title: Thread Pool
description: Thread Pool
header: Thread Pool
tags: [Java, Thread, Thread Pool]
---

## 정의

Java Thread Pool은 Worker Thread의 Pool을 관리한다. 실행 대기 중인 작업을 보관하는 큐(Queue)를 포함한다.
`ThreadPoolExecutor`를 이용하여 Thread Pool을 생성한다.

Java Thread Pool은 Runnable Thread의 컬렉션을 관리한다. Worker Thread는 큐에서 Runnable을 실행한다.
java.util.concurrent.Executors에서 factory를 제공하고, 
java.util.concurrent.Executor 인터페이스를 위한 메소드를 지원해서 Thread Pool을 생성한다.

Executor는 유틸리티 클래스로 Factory 메소드를 통해 `ExecutorService`, `ScheduledExecutorService`, `ThreadFactory`, `Callable` 클래스와 함께 유용한 메소드 실행을 지원한다. 

WorkerThread.java - Runnable.class 활용
```java
public class WorkerThread implements Runnable {
  
    private String command;
    
    public WorkerThread(String s) {
        this.command=s;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
        processCommand();
        System.out.println(Thread.currentThread().getName()+" End.");
    }

    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString(){
        return this.command;
    }
}

```


## ExecutorService 예제

SimpleThreadPool.java - Executors 프레임워크로부터 고정된 Thread Pool을 만든다.
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SimpleThreadPool {

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);
          }
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

위 프로그램은 Thread Pool 사이즈를 5개의 Worker Thread로 고정하여 만들었다. 그리고 10개의 Job을 Pool에 넘긴다.
Pool 사이즈가 5개이므로 5개가 넘어가면 선행된 Job이 끝날 때까지 기다린다. 큐에 있는 Job은 Worker Thread가 꺼내 작업하게 된다.

```bash
pool-1-thread-2 Start. Command = 1
pool-1-thread-4 Start. Command = 3
pool-1-thread-1 Start. Command = 0
pool-1-thread-3 Start. Command = 2
pool-1-thread-5 Start. Command = 4
pool-1-thread-4 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
pool-1-thread-3 End.
pool-1-thread-3 Start. Command = 8
pool-1-thread-2 End.
pool-1-thread-2 Start. Command = 9
pool-1-thread-1 Start. Command = 7
pool-1-thread-5 Start. Command = 6
pool-1-thread-4 Start. Command = 5
pool-1-thread-2 End.
pool-1-thread-4 End.
pool-1-thread-3 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
Finished all threads
```

Pool 사이즈 만큼 1~5번 Thread가 실행되고 나머지 Job은 종료되는 Thread 이후에 실행 된 것을 볼 수 있다.


## ThreadPoolExecutor 예

`Executors` 클래스에서 `ThreadPoolExecutor`를 사용하여 `ExecutorService`의 간단한 구현을 제공한다.
ThreadPoolExecutor는 더 많은 기능을 제공한다.
ThreadPoolExecutor 인스턴스를 생성할 때 사용할 스레드 수를 지정할 수도 있다.
Thread Pool 크기를 제한하고 Worker 큐에 맞지 않는 작업을 처리하기 위해 자체적으로 RejectedExecutionHandler 구현을 생성할 수 있다. 

```java
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;

public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {

    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println(r.toString() + " is rejected");
    }

}
```

`ThreadPoolExecutor`는 현재 상태, Pool 크기, 활성 Thread 수 및 작업 수를 확인할 수 있는 여러 메소드를 제공한다.
그래서 일정한 간격으로 Executor 정보를 출력하는 모니터 Thread가 있다.

```java
import java.util.concurrent.ThreadPoolExecutor;

public class MyMonitorThread implements Runnable {
    private ThreadPoolExecutor executor;
    private int seconds;
    private boolean run = true;

    public MyMonitorThread(ThreadPoolExecutor executor, int delay) {
        this.executor = executor;
        this.seconds = delay;
    }
    
    public void shutdown() {
        this.run = false;
    }
    
    @Override
    public void run() {
        while (run) {
            System.out.println(
                String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
                    this.executor.getPoolSize(),
                    this.executor.getCorePoolSize(),
                    this.executor.getActiveCount(),
                    this.executor.getCompletedTaskCount(),
                    this.executor.getTaskCount(),
                    this.executor.isShutdown(),
                    this.executor.isTerminated()));
            try {
                Thread.sleep(seconds*1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
            
    }
}
```

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class WorkerPool {

    public static void main(String args[]) throws InterruptedException {
        //RejectedExecutionHandler implementation
        RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
        //Get the ThreadFactory implementation to use
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //creating the ThreadPoolExecutor
        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
        //start the monitoring thread
        MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
        Thread monitorThread = new Thread(monitor);
        monitorThread.start();
        
        //submit work to the thread pool
        for (int i=0; i<10; i++) {
            executorPool.execute(new WorkerThread("cmd"+i));
        }
        
        Thread.sleep(30000);
        //shut down the pool
        executorPool.shutdown();
        //shut down the monitor thread
        Thread.sleep(5000);
        monitor.shutdown();
    }
}
```

ThreadPoolExecutor를 초기화하는 동안 최초 Pool 크기를 2, 최대 크기를 4, 작업 큐 크기를 2로 유지한다는 점을 주의해야 한다.
따라서 실행중인 작업이 4개고 더 많은 작업이 실행되면 작업 큐는 2개만 포함되며 나머지 작업은 `RejectedExecutionHandlerImpl`에 의해 처리된다.

아래와 같은 실행결과를 볼 수 있다.

```bash
pool-1-thread-1 Start. Command = cmd0
pool-1-thread-4 Start. Command = cmd5
cmd6 is rejected
pool-1-thread-3 Start. Command = cmd4
pool-1-thread-2 Start. Command = cmd1
cmd7 is rejected
cmd8 is rejected
cmd9 is rejected
[monitor] [0/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-4 End.
pool-1-thread-1 End.
pool-1-thread-2 End.
pool-1-thread-3 End.
pool-1-thread-1 Start. Command = cmd3
pool-1-thread-4 Start. Command = cmd2
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-4 End.
[monitor] [4/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
```

Notice the change in active, completed and total completed task count of the executor. 
We can invoke shutdown() method to finish execution of all the submitted tasks and terminate the thread pool.



> 참고<br/>
> https://www.journaldev.com/1069/threadpoolexecutor-java-thread-pool-example-executorservice