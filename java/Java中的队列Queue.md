## 简介

Queue（队列）是一种先进先出（FIFO）的有序表。 在 Java 中定义了 java.util.Queue<E> 接口来表示队列。Queue<E> 和 List<E>、Set<E> 属于同一级别的接口，都继承自 Collection<E>。

> FIFO：first in first out

## 源码

Queue<E> 源码如下：

```java
public interface Queue<E> extends Collection<E> {
    //向队列中插入一个元素，并返回true
    //如果队列已满，抛出IllegalStateException异常
    boolean add(E e);

    //向队列中插入一个元素，并返回true
    //如果队列已满，返回false
    boolean offer(E e);

    //取出队列头部的元素，并从队列中移除
    //队列为空，抛出NoSuchElementException异常
    E remove();

    //取出队列头部的元素，并从队列中移除
    //队列为空，返回null
    E poll();

    //取出队列头部的元素，但并不移除
    //如果队列为空，抛出NoSuchElementException异常
    E element();

    //取出队列头部的元素，但并不移除
    //队列为空，返回null
    E peek();
}
```

从源码中得知，我们可以把方法分为三类：插入（add、offer）、获取队列头部元素并删除（remove、poll）、获取队列头部元素不删除（element、peek）。

| 方法       | 返回       | 异常                                         |
| ---------- | ---------- | -------------------------------------------- |
| add(E e)   | true       | 如果队列已满，抛出IllegalStateException异常  |
| offer(E e) | true/false |                                              |
| remove()   | E          | 如果队列已满，抛出NoSuchElementException异常 |
| poll()     | E          | 队列为空，返回 null                          |
| element()  | E          | 如果队列为空，抛出NoSuchElementException异常 |
| peek()     | E          | 队列为空，返回 null                          |

## 队列的分类

### 阻塞队列

>  所谓的阻塞队列就是：1、当队列为空，某个线程去消费队列，这个线程就会变成阻塞状态直到有其他线程操作队列使用不为空。2、当队列是满的时候，某个线程去操作队列插入元素，这个线程就会变成阻塞状态直到有其他线程操作队列使其有空闲位置。

在 Java 中定义来 BlockingQueue<E> 接口来表示阻塞队列，在 java.util.concurrent 包下。下面我们将介绍 BlockingQueue<E> 的5个实现类。

#### ArrayBlockingQueue<E>

ArrayBlockingQueue 的底层数据结构为数组，可见部分源码：

```java
		/** The queued items */
    final Object[] items;

    /**
     * Creates an {@code ArrayBlockingQueue} with the given (fixed)
     * capacity and default access policy.
     *
     * @param capacity the capacity of this queue
     * @throws IllegalArgumentException if {@code capacity < 1}
     */
    public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }

    /**
     * Creates an {@code ArrayBlockingQueue} with the given (fixed)
     * capacity and the specified access policy.
     *
     * @param capacity the capacity of this queue
     * @param fair if {@code true} then queue accesses for threads blocked
     *        on insertion or removal, are processed in FIFO order;
     *        if {@code false} the access order is unspecified.
     * @throws IllegalArgumentException if {@code capacity < 1}
     */
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }
```

由上面可以看出，ArrayBlockingQueue 是一个内部由数组支持的有界队列。在构造时，我们一定要指定容量。

[API](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html)

#### LinkedBlockingQueue<E>

LinkedBlockingQueue 一个内部由链接节点支持的可选有界队列。初始化时不需要指定队列的容量，默认是 Integer.MAX_VALUE，也可以看成容量无限大。此队列按 FIFO（先进先出）排序元素 。

[API](https://www.matools.com/api/java8)

#### PriorityBlockingQueue<E>

PriorityBlockingQueue 一个内部由优先级堆支持的无界优先级队列，其实就是 PriorityQueue 的加锁线程安全版。[PriorityQueue 介绍](#PriorityQueue)

[API](https://www.matools.com/api/java8)

#### DelayQueue<E>

DelayQueue一个内部由优先级堆支持的、基于时间的调度队列。

#### SynchronousQueue<E>

### 非阻塞队列

#### PriorityQueue

关于 PriorityQueue 的学习是来自[廖雪峰-PriorityQueue](https://www.liaoxuefeng.com/wiki/1252599548343744/1265120632401152)。

我在这里模仿举一个银行办理业务的例子。现在有 10 个人在银行取票等待办理业务，这 10 个人肯定是按照取票的顺序办理的，此时我们使用一般的 Queue 就可以完成，假如这时候来了一个特殊的人员需要办理业务，我们肯定需要给他优先办理，这时候就需要引入 PriorityQueue 队列了，PriorityQueue 支持元素的优先级，如以下代码示例：

```mysql

```



