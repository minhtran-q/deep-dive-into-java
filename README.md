# Deep dive into java
## Stream
## Concurrency
### Concurrency vs Parallelism
### Race condition
The situation where two or more threads complete the same resource, where the sequence in which resource is accessed is significant, is called **`race condition`**

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
