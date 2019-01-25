---
title: Thread Safety
description: Thread Safety
header: Thread Safety
tags: [Java, Thread, Thread Safety, Synchronized, Concurrency]
---

Thread Safety는 매우 중요한 부분이다.
Java가 Thread를 이용하여 Multi Thread 환경을 제공한다.
동일한 객체에서 생성된 Multi Thread는 객체 변수를 공유하며, 
Thread가 공유 데이터를 읽고 업데이트 하는데 사용될 때 데이터 불일치로 이어질 수 있다.

## Thread Safety

데이터 불일치가 생기는 이유는 업데이트하는 모든 데이터가 Atomic 프로세스가 아니기 때문이다. 다음 3단계가 필요하다.
1. 값을 읽는다.
2. 필요한 연산을 수행하며 업데이트된 값을 얻는다.
3. 업데이트된 값을 필드 참조에 할당한다.

```java
public class ThreadSafety {

    public static void main(String[] args) throws InterruptedException {
    
        ProcessingThread pt = new ProcessingThread();
        Thread t1 = new Thread(pt, "t1");
        t1.start();
        Thread t2 = new Thread(pt, "t2");
        t2.start();
        //wait for threads to finish processing
        t1.join();
        t2.join();
        System.out.println("Processing count=" + pt.getCount());
    }

}

class ProcessingThread implements Runnable {
    private int count;
    
    @Override
    public void run() {
        for(int i=1; i < 5; i++){
            processSomething(i);
        	count++;
        }
    }

    public int getCount() {
        return this.count;
    }

    private void processSomething(int i) {
        // processing some job
        try {
            Thread.sleep(i * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
}
```

위 코드는 2개의 Thread로 반복하며, 카운트는 1씩 증가를 4차례하게 된다. Thread 실행이 끝나면 결과는 8이된다.
하지만 위 코드를 여러번 실행했을 때, 결과 값은 6, 7, 8 등으로 나오는 것을 볼 수 있을 것이다.
이런 일이 생기는 것은 `count++` 구문이 Atomic 동작처럼 보일지 몰라도 데이터 왜곡이 발생한다. 


## Thread Safety in Java

Thread Safety는 프로세스(Process)를 멀티스레드(Multi Thread) 환경에서 사용되는 프로그램을 안전하게 만들어 준다.
또한, 프로그램 Thread를 안전하게 만들 수 있는 다양한 방법이 있다.

 - Synchronization은 Thread Safety를 위한 매우 쉽고, 널리 활용되는 툴이다. 
 - java.util.concurrent.atomic 패키지에서 나온 Atomic Wrapper 클래스 사용할 수 있다. 예를들어 AtomicInteger.
 - java.util.concurrent.locks 패키지는 잠금(Lock)기능을 사용할 수 있다.
 - Thread Safe 컬랙션 클래스 사용하여 Thread의 안전을 위한 ConcurrentHashMap 사용여부를 확인한다. 
 - 변수와 함께 휘발성 키워드를 사용하여 모든 Thread가 Thread Cache에서 읽지 않고 메모리의 데이터를 읽도록 한다.

#### Synchronized

`synchronized`를 활용하여 잘못 증가되는 카운트를 바로잡을 수 있다.

```java
//dummy object variable for synchronization
private Object mutex=new Object();
...
//using synchronized block to read, increment and update count value synchronously
synchronized (mutex) {
        count++;
}
```

아래 예제를 참고해 보자.

```java
public class MyObject {
  // Locks on the object's monitor
  public synchronized void doSomething() { 
    // ...
  }
}
 
// Hackers code
MyObject myObject = new MyObject();
synchronized (myObject) {
  while (true) {
    // Indefinitely delay myObject
    Thread.sleep(Integer.MAX_VALUE); 
  }
}
```

코드가 myObject 인스턴스를 잠그려고 하는 것을 주목하고, 일단 잠금을 설정하면, 
잠금 장치 대기 중 `Doomething()` 메서드를 차단하는 건 아니지만 
이로 인해 시스템이 교착 상태(Deadlock)가 되고 서비스 거부(DoS)가 발생할 수 있다.

```java
public class MyObject {
  public Object lock = new Object();
 
  public void doSomething() {
    synchronized (lock) {
      // ...
    }
  }
}

//untrusted code

MyObject myObject = new MyObject();
//change the lock Object reference
myObject.lock = new Object();
```

lock 객체가 public 이며, 참조를 변경하여 Multi Thread에 의해 동시에 동기화된 블록을 실행할 수 있다.
private 객체가 있다면 비슷한 케이스가 있을 수 있다. 하지만 참조를 변경할 수 있는 setter 메소드도 있다.


```java
public class MyObject {
  //locks on the class object's monitor
  public static synchronized void doSomething() { 
    // ...
  }
}
 
// hackers code
synchronized (MyObject.class) {
  while (true) {
    Thread.sleep(Integer.MAX_VALUE); // Indefinitely delay MyObject
  }
}
```

위의 `hackers code`가 클래스 모니터를 잠그고 해제하지 않으면 시스템은 교착상태에 빠져질 수 있다는 점을 주의하자.

<br/>

Multi Thread가 동일한 String 배열에서 작동하고 한 번만 처리되면 Thread 이름을 배열 값에 추가하는 예제는 다음과 같다. 

```java
import java.util.Arrays;

public class SyncronizedMethod {

    public static void main(String[] args) throws InterruptedException {
        String[] arr = {"1","2","3","4","5","6"};
        HashMapProcessor hmp = new HashMapProcessor(arr);
        Thread t1=new Thread(hmp, "t1");
        Thread t2=new Thread(hmp, "t2");
        Thread t3=new Thread(hmp, "t3");
        long start = System.currentTimeMillis();
        //start all the threads
        t1.start();t2.start();t3.start();
        //wait for threads to finish
        t1.join();t2.join();t3.join();
        System.out.println("Time taken= "+(System.currentTimeMillis()-start));
        //check the shared variable value now
        System.out.println(Arrays.asList(hmp.getMap()));
    }

}

class HashMapProcessor implements Runnable{
    
    private String[] strArr = null;
    
    public HashMapProcessor(String[] m){
        this.strArr=m;
    }
    
    public String[] getMap() {
        return strArr;
    }

    @Override
    public void run() {
        processArr(Thread.currentThread().getName());
    }

    private void processArr(String name) {
        for(int i=0; i < strArr.length; i++){
            //process data and append thread name
            processSomething(i);
            addThreadName(i, name);
        }
    }
    
    private void addThreadName(int i, String name) {
        strArr[i] = strArr[i] +":"+name;
    }

    private void processSomething(int index) {
        // processing some job
        try {
            Thread.sleep(index*1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
}
```

위 코드의 결과는 아래와 같다.

```bash
Time taken= 15005
[1:t2:t3, 2:t1, 3:t3, 4:t1:t3, 5:t2:t1, 6:t3]
```

String 배열 값은 데이터를 공유하므로 데이터가 손상되고 동기화가 동작하지 않는다. 
안전하게 동작하는 Thread를 만들기 위해 `addThreadName()` 메소드를 아래와 같이 수정할 수 있다.

```java
private Object lock = new Object();
    private void addThreadName(int i, String name) {
        synchronized(lock){
        strArr[i] = strArr[i] +":"+name;
    }
}
```

수정된 코드는 아래와 같이 정확하게 동작하는 것을 볼 수 있다.

```bash
Time taken= 15004
[1:t1:t2:t3, 2:t2:t1:t3, 3:t2:t3:t1, 4:t3:t2:t1, 5:t2:t1:t3, 6:t2:t1:t3]
```

이것이 Thread Safety의 모든 것이다. :)

> 참고<br/>
> https://www.journaldev.com/1061/thread-safety-in-java
