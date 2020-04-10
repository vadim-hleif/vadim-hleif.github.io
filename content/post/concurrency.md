---
date: 2020-04-02
title: "Concurrency in Java"
---
Let's talk about concurrency in java.
There are two options to deal with concurrency 
* Optimistic lock
* Pessimistic lock

****
#### 1. Optimistic lock

In this strategy, you will try to save record when previous state of the record as you expected.

Example: 

**CAS** -- compare and swap or compare and set. Processor instruction which guarantees reading, comparing and writing as atomic operation. 

```java
 variable.cas(expectedValue, newValue)
```

Java has AtomicReference class for objects and also has similar wrappers for many primitives. 

Popular example with money transfer:
```java
public void transfer(Account source, Account target, int amount) {
    int money;
    do {
        //A: get value from AtomicInteger
        money = source.getMoney().get(); 
    }
    //B: code tries to set new value only if there aren't any changes 
    while (!source.getMoney().compareAndSet(money, money - amount)); 
```
> AtomicInteger has **getAndAdd** method, code above is only a simple example.

It's so easy -- we try to set new balance only when previous one hasn't changed. And if balance has changed after **A** part and
before **B** part **compareAndSet** won't change anything, will return false and cycle will try again and again until success.
There is one possible overhead -- calculating new value every time after failure. If we speak about real applications
with microservices and database approach will be the same -- try to save only when there are no changes:

```sql
UPDATE account
SET amount = amount - 100
AND version = 2
WHERE id = 1
  AND version = 1
```
And when it is network access to some web service or database call - retry in optimistic lock can be so costly.

Also it won't be so easy to implement optimistic lock, if to speak about more complex sample:

```java
public class LinkedQueue<E> {

  private AtomicReference<Node<E>> tail
      = new AtomicReference<>(new Node<E>(null, null));

  public boolean put(E item) {
    Node<E> newNode = new Node<>(item, null);

    while (true) {
      Node<E> current = tail.get();
      Node<E> next = current.next.get();

      if (current == tail.get()) {
        if (next == null) /* A */ {
          if (current.next.compareAndSet(null, newNode)) /* C */ {
            tail.compareAndSet(current, newNode) /* D */;
            return true;
          }
        } else {
          tail.compareAndSet(current, next) /* B */;
        }
      }
    }
  }
```

So, here is more complex case with double-variable needed synchronization - **tail** and reference from previous node **next**.
There is one possible problem - when we changed next link **C** but didn't change tail **D**, in other words - full operation isn't completed.
To avoid this problem there is additional check - **A**, when current thread detect, that another thread currently is working, current thread will help to finish operation -
**B**. Code from **B** does the same thing as code in **D**, so any other thread can help with finishing current operation and then try to do its operation.

#### 2. Pessimistic lock

Another way is locking resources or synchronizing parallel access. There are a few new terms:
* Semaphore - counter which controls count of threads which can have access to resource
* Mutex - semaphore with initial value **1**
* Monitor - entity which provides high level API for synchronization, implemented over semaphore. Example in java 
is keyword **synchronized**

So synchronization is a way to ordered access to resources - when one thread use some resource, 
others can't do anything with resource. Every object in Java has personal mutex, so when we use keyword **synchronized** on some object method,
we use monitor which locks current object and all methods with **synchronized** keyword can't be called while method is working.

If try to use such approach in money transfer case we will have one problem - possible deadlock. It will happen when some thread tries to transfer
from **first** to **second**, and another tries to transfer from **second** to **first**.

```java
public void transfer(Account source, Account target, int amount) {
  synchronized (source) {
    synchronized (target) {
      source.setMoney(source.getMoney() - amount);
      target.setMoney(target.getMoney() + amount);
    }
  }
}
```
To avoid this problem we can use something, which will guarantee the order - sort account by some fields, for example id.
```java
public void transfer(Account source, Account target, int amount) {
  Account first = source.getId() < target.getId() ? source : target;
  Account second = source.getId() < target.getId() ? target : source;
  synchronized (first) {
    synchronized (second) {
      source.setMoney(source.getMoney() - amount);
      target.setMoney(target.getMoney() + amount);
    }
  }
}
```

#### 3. Which is better?

There isn’t rule like Amdahl’s Law which will guarantee the better option.
Under high contention -- lock-based algorithms can offer better performance, in some cases - non blocking algorithms are better.
But there aren't any digits - just empirical way

****
#### Links:
- https://www.ibm.com/developerworks/java/library/j-jtp11234/
- https://www.ibm.com/developerworks/java/library/j-jtp10264/
- https://www.ibm.com/developerworks/java/library/j-jtp04186/