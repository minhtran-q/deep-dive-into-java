# Deep dive into java
## Stream
## Concurrency
### Concurrency vs Parallelism

+ <details>
  <summary>Concurrency</summary>
  
  ![](images/concurrency.PNG)
  
  Concurrency means that an application is making progress on more than one task - at the same time or at least seemingly at the same time. If the computer only has one CPU the application may not make progress on more than one task at _exactly the same time_
  
</details>

+ <details>
  <summary>Parallel</summary>

  ![](images/parallel.PNG)
  
  Parallel execution is when a computer has more than one CPU or CPU core, and makes progress on more than one task simultaneously.
  
</details>

+ <details>
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
  <summary>wait(), notify() and notifyAll()</summary>
  
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
  
### Lock and Synchronized Block
  
<details>
  <summary>Differences are as follows: </summary>
  
  + lock() & unlock() operation in separate methods
  + Support fairness by specifying the fairness property
  + 
  
</details>
  
## Exception Handling
### Checked exception & Unchecked exception
  
<details>
  <summary>When to choose checked and unchecked exceptions</summary>
  
  
  
</details>
