# Deep dive into java
## Stream
## Concurrency
### Concurrency vs Parallelism

<details>
  <summary>Concurrency</summary>
  
  ![](images/concurrency.PNG)
  
  Concurrency means that an application is making progress on more than one task - at the same time or at least seemingly at the same time. If the computer only has one CPU the application may not make progress on more than one task at _exactly the same time_
  
</details>

<details>
  <summary>Parallel</summary>

  ![](images/parallel.PNG)
  
  Parallel execution is when a computer has more than one CPU or CPU core, and makes progress on more than one task simultaneously.
  
</details>

<details>
  <summary>Parallel Concurrency</summary>

  ![](images/concurrency-parallel.PNG)
  
</details>

### Race condition
`Race conditions` occur only if multiple threads are accessing the same resource, and one or more of the threads write to the resource. If multiple threads read the same resource `race conditions` do not occur.

Two types of `race condition`:
+ <details>
  <summary>Read-modify-write</summary>
  
  ```
  public class Counter {
       protected long count = 0;

       public void add(long value){
           this.count = this.count + value;
       }
  }
  ```
  For example, two threads wanted to add values 2 and 3. Thus the result should be 5 after the two threads complete execution. In the above case it is 2, but it could as well have been 3.
</details>

+ <details>
  <summary>Check-then-act</summary>
  
  ```
  public class CheckThenActExample {

      public void checkThenAct(Map<String, String> sharedMap) {
          if(sharedMap.containsKey("key")){
              String val = sharedMap.remove("key");
              if(val == null) {
                  System.out.println("Value for 'key' was null");
              }
          } else {
              sharedMap.put("key", "value");
          }
      }
  }
  ```
</details>
  
### Java memory model
<details>
  <summary>Bridging The Gap Between The Java Memory Model And The Hardware Memory Architecture</summary>

  ![](images/hardware.PNG)
  
</details>

### Thread Signaling
  
<details>
  <summary>Overview</summary>

  ![](images/waitnotify.png)
  
</details>

<details>
  <summary>wait(), notify() and notifyAll()</summary>

  + **wait()**<br/>
  
    It tells the calling thread to give up the lock and go to sleep until some other thread enters the same monitor and calls notify().
  
  + **notify()**<br/>
  
    It wakes up one single thread that called wait() on the same object. It should be noted that calling notify() does not actually give up a lock on a resource. It tells a waiting thread that that thread can wake up.
  
  + **notifyAll()**<br/>
  
    It wakes up all the threads that called wait() on the same object.
  <br/>
  _main_

  ```
  public class Hello {
      public static void main(String[] args) {
        Queue < String > q = new LinkedList < > ();
        boolean exit = false;
        Producer p = new Producer(q, exit);
        p.start();
        Consumer c = new Consumer(q, exit);
        c.start();
      }
  }
  ```


  _producer_
  ```
  public class Producer extends Thread {

      private volatile Queue < String > sharedQueue;

      private volatile boolean bExit;

      public Producer(Queue < String > myQueue, boolean bExit) {
          this.sharedQueue = myQueue;
          this.bExit = bExit;
      }
      public void run() {
          while (!bExit) {
              synchronized(sharedQueue) {
                  while (sharedQueue.isEmpty()) {
                    String item = String.valueOf(System.nanoTime());
                    sharedQueue.add(item);
                    System.out.println("Producer added : " + item);
                      try {
                          System.out.println("Producer sleeping by calling wait: " + item);
                          sharedQueue.wait();
                          System.out.println("Producer wake up: ");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
              }
          }
      }
  }
  ```

  _consumer_

  ```
  public class Consumer extends Thread {

      private volatile Queue < String > sharedQueue;

      private volatile boolean bExit;

      public Consumer(Queue < String > myQueue, boolean bExit) {
          this.sharedQueue = myQueue;
          this.bExit = bExit;
      }
      public void run() {
          while (!bExit) {
              synchronized(sharedQueue) {
                  while (!sharedQueue.isEmpty()) {
                      String item = sharedQueue.poll();
                      System.out.println("Consumer removed : " + item);
                      System.out.println("Consumer notifying Producer: " + item);
                      sharedQueue.notify();
                  }
              }
          }
      }
  }
  ```
  _Output_

  ```
    Producer added : 12275948008616
    Producer sleeping by calling wait: 12275948008616
    Consumer removed : 12275948008616
    Consumer notifying Producer: 12275948008616
    Producer wake up: 
    Producer added : 12275948047960
    Producer sleeping by calling wait: 12275948047960
    Consumer removed : 12275948047960
    Consumer notifying Producer: 12275948047960
  ```

  + Ref: https://www.java67.com/2019/05/when-and-how-to-use-wait-and-notify-in-Java.html
  + Ref: https://ducmanhphan.github.io/2019-12-07-Using-wait-notify-in-synchronized-method-block-of-Multithreading-Java/
  + Ref: https://howtodoinjava.com/java/multi-threading/wait-notify-and-notifyall-methods/
  </details>

### Java monitor
<details>
  <summary>What is a monitor?</summary>
  
  Simply put, a _**monitor**_ is something that a thread can grab and hold, preventing all other threads from grabbing that same monitor and forcing them to wait until the monitor is released. 
</details>
<details>
  <summary>monitor vs synchronized keyword</summary>
  
  ```
  Object foo = new Object();
  synchronized (foo) {
    System.out.println("Hello world.");
  }
  ```
  The current thread will first grab the monitor associated with the object stored in variable `foo` and hold it while it prints `"Hello world"`, then releases it.
  
</details>
  
### Lock
  
<details>
  <summary>Lock and Synchronized Block</summary>
  
  + _lock()_ & _unlock()_ operation in separate methods
  + Support fairness by specifying the fairness property
  + The Lock API provides _tryLock()_ method. This reduces blocking time of thread waiting for the lock
  + Provides _lockInterruptibly()_ which can be used to interrupt the thread when it's waiting for the lock
  
  Ref: https://www.baeldung.com/java-concurrent-locks
  Ref: https://winterbe.com/posts/2015/04/30/java8-concurrency-tutorial-synchronized-locks-examples/
  
</details>
  
<details>
  <summary>Condition Variable</summary>
  
  Condition variables are instance of java.util.concurrent.locks.Condition class. As a reentrant lock replaces synchronized blocks or methods, a condition replaces the object monitor methods. which provides inter-thread communication methods similar to `wait()`, `notify()` and `notifyAll()` e.g. `await()`, `signal()`, and `signalAll()`.
  
  _Note: The lock can have multiple condition variables associated with the same lock_
  
  + Ref: https://javarevisited.blogspot.com/2015/06/java-lock-and-condition-example-producer-consumer.html#ixzz7QhNth9mS
  + Ref: https://medium.com/@tarunjain07/volatile-reentrant-lock-vs-synchronize-condition-variable-66e870a8434d#2738
  
</details>
<details>
  <summary>Pros & cons</summary>
  
  Pros:
  1) Ability to lock interruptibly.
  2) Ability to timeout while waiting for lock.
  3) Power to create fair lock.
  4) API to get list of waiting thread for lock.
  5) Flexibility to try for lock without blocking.
  
  Cons:
  1) wrapping method body inside `try-finall` block

Read more: https://javarevisited.blogspot.com/2013/03/reentrantlock-example-in-java-synchronized-difference-vs-lock.html#ixzz7QhMEYXjR
  
  Ref: https://javarevisited.blogspot.com/2013/03/reentrantlock-example-in-java-synchronized-difference-vs-lock.html#axzz7Qc6BsHhk
  
</details>
<details>
  <summary>ReadWriteLock vs ReentrantLock</summary>
  
  **ReadWriteLock**:
  + `ReadLock` and `WriteLock` though are 2 separate instances only 1 will be allowed at a time...
  + Either `ReadLock` is being used _**(by n threads)**_ or `WriteLock` is being used _**(by 1 thread)**_
  + But never both at same time
  
  
  
  Ref: https://www.youtube.com/watch?v=7VqWkc9o7RM&ab_channel=DefogTech
</details>

  
### Semaphores
  A Semaphore in Java is a Thread Synchronization construct that controls access to the shared resource with the help of counters. 
  
<details>
  <summary>Types of semaphores</summary>
  
  + Counting Semaphores
  + Bounded Semaphores
  + Timed Semaphores
  + Timed Semaphores
  
</details>
  
<details>
  <summary>Semaphore vs Mutex</summary>
    
  Semaphore can be counted, while mutex can only count to 1.
  
  _Note: Mutex (synchronized keyword, ReentrantLock, ...)_
  
  Ref: https://www.baeldung.com/java-mutex
</details>
  
## Exception Handling
### Checked exception & Unchecked exception
  
<details>
  <summary>When to choose checked and unchecked exceptions</summary>
  
  
  
</details>
