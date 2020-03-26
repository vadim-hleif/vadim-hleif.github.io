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
And when it will network access to some webservice - retry can be co costly

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

Для счетчика или стека (когда надо обновить только одно значение) реализация неблокирующей синхронизации простая, но, когда надо обновить несколько значений реализация значительно усложняется - как например в листинге ниже.

Решается благодаря тому, что другой поток знает, как перевести текущее состояние очереди из незавершенного состояния, оставленного другим потоком (после выполнения C и до выполнения D) в завершенное (B == D). 

 

✓	lock without thread sleep (все ждут пока счетчик не станет равен 0)
 
 


Optimistic lock vs pessimistic lock
https://stackoverflow.com/a/58952004/10758502

тоже самое как в CAS – меняем только если видим версию, которую мы ожидаем, может быть хуже, когда параллельных запросов очень много, часто случаются ошибки и приходится откатывать много действий – тогда pessimistic (с локами) будет лучше по производительности.






















✓	algorithms cost and sort

https://ru.wikipedia.org/wiki/Быстрая_сортировка
https://www.daniweb.com/programming/computer-science/threads/13488/time-complexity-of-algorithm

лучший алгоритм, используется в jdk для сортировки. Вообще “лучшего” алгоритма не существует, все зависит от входных данных – e.g.

The time complexity of Quicksort is O(n log n) in the best case, O(n log n)in the average case, and O(n^2) in the worst case. But because it has the best performance in the average case for most inputs, Quicksort is generally considered the “fastest” sorting algorithm. 

            
Основание у log не пишут, так как от него не зависит во сколько раз станет выше сложность.

O(n log n)  - при маленьких значениях видно, O(n) легче, при больших наоборот. 
O(log n)     - видно что величина прироста постоянна (бинарный поиск пример)
