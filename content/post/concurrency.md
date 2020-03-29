---
date: 2020-03-26
title: "Concurrency in Java"
---
Let's talk about concurrency in java.
There are two options to deal with concurrency 
* Optimistic lock
* Pessimistic lock

****
#### 1. Optimistic lock

In this strategy, you will try to save record only when previous state as you expected.

Example: 

CAS -- compare and swap or compare and set. Processor instruction which guarantees reading, comparing and writing as atomic operation. 

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
    while (source.getMoney().compareAndSet(money, money - amount)); 
```
> AtomicInteger has **getAndAdd** method, code above only simple example.

It's so easy -- we try to set new balance only when previous one didn't change. And if balance changed after **A** part and
before **B** part **compareAndSet** doesn't change anything, returns false and cycle will try again and again until success.
There is one possible overhead -- calculating new value every time after failure. If we will speak about real applications
with microservices and database approach is the same -- try to save only when there aren't any changes:

```sql
UPDATE account
SET amount = amount - 100
AND version = 2
WHERE id = 1
  AND version = 1
```
And when it will network access to some webservice or database call - optimistic lock can be so costly.

Also to implement optimistic lock won't be so easy, if speak about more complex sample 

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

#### 2. Pessimistic lock

Another way is locking resources or synchronize parallel access. There are few new terms:
* Semaphore - counter which controls count of threads which can have access to resource
* Mutex - semaphore with initial value **1**
* Monitor - entity which provides high level API for synchronization, implemented over semaphore. Example in java 
is keyword **synchronized**

So synchronization is a way to ordered access to resources - when one thread use some resource, 
others can't do anything with it. Every object in Java has personal mutex, so when we use keyword **synchronized** on some object method,
we use monitor which locks current object and others methods with **synchronized** keyword can't be called while method is working.

****

Under high contention -- when many threads are pounding on a single memory location -- lock-based algorithms start to offer better throughput than nonblocking ones because when a thread blocks, it stops pounding and patiently waits its turn, avoiding further contention.


Источники:
•	https://www.ibm.com/developerworks/java/library/j-jtp11234/
•	https://www.ibm.com/developerworks/java/library/j-jtp10264/
•	https://www.ibm.com/developerworks/java/library/j-jtp04186/

углубление:
Amdahl’s Law. 1 / (F+ ((1-F)/n)). https://ru.wikipedia.org/wiki/Закон_Амдала
aba problem https://en.wikipedia.org/wiki/ABA_problem
aba может возникнуть из-за каких-нибудь сайд эффектов помимо просто изменения значения с B -> A 

Сложный кейс, когда надо сделать две операции:


Optimistic lock vs pessimistic lock
https://stackoverflow.com/a/58952004/10758502




















✓	algorithms cost and sort

https://ru.wikipedia.org/wiki/Быстрая_сортировка
https://www.daniweb.com/programming/computer-science/threads/13488/time-complexity-of-algorithm

лучший алгоритм, используется в jdk для сортировки. Вообще “лучшего” алгоритма не существует, все зависит от входных данных – e.g.

The time complexity of Quicksort is O(n log n) in the best case, O(n log n)in the average case, and O(n^2) in the worst case. But because it has the best performance in the average case for most inputs, Quicksort is generally considered the “fastest” sorting algorithm. 

            
Основание у log не пишут, так как от него не зависит во сколько раз станет выше сложность.

O(n log n)  - при маленьких значениях видно, O(n) легче, при больших наоборот. 
O(log n)     - видно что величина прироста постоянна (бинарный поиск пример)
