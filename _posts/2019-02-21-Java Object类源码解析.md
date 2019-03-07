---
title: Java Object类源码解析
description: 
    - 本文主要介绍了
categories: Java
tags: 
    - Java
    - 基础
---
 
#### 1.介绍
---
Object 作为类层次结构中的根类，是每一个类的父类。Object 类中的方法可以说是最基础最基本的方法，类中的每一个方法我们都应该特别熟悉了解。这些方法包括类的信息获取、对象的之间的关系判断、对象的复制、以及对象的同步机制等最为基础的方面。
在创建新类时，如果不显示的指明继承的类，都是默认继承 Object 类的。

#### 2.方法列表
---
方法名 | 访问权限 | 说明
--- | --- | ---
getClass | public,final,native | 返回当前对象的运行时 Class 对象。
hashCode | public,native | 返回当前对象的哈希值,该方法有效的支持了 Map 可以用对象作为 key。
equals | public |判断当前对象与参数传入的对象是否相等，默认是判断引用是否相等。
clone | protected,native | 创建并返回当前对象的复制。
toString | public | 返回当前对象的字符串展现，为了方便阅读，官方推荐所有子类重写该方法。
notify | public,final,native | 唤醒一个在当前对象的 waitSet 中等待的线程。
notifyAll | public,final,native | 唤醒所有在当前对象的 waitSet 中等待的线程。
wait | public,final,native | 使当前线程进入 monitor 的等待集合中，当前线程必须已经获取到 monitor。
finalize | protected | 垃圾回收器在判断该对象没有引用指向后，会调用该方法，子类通常重写该方法去处理额外的资源释放。 

如上表所示，Object 类中主要包含了这 9 个方法。理论上说Java中的任何类都可以调用这 9 个方法，这 9 个方法可以看作是所有类共同拥有的最基本的方法。

#### 3.源码解析
---
##### 3.1 getClass
源代码：
```java
public final native Class<?> getClass();
```
native 关键词表明这是一个本地化方法，在本地库中用非 java 语言实现的。
Java 通过该方法实现了反射机制，该方法返回当前对象的 Class 对象，通过 Class 对象可以在运行时中就可以获取该类的所有属性与方法。
##### 3.2 hashCode
源代码：
```java
public native int hashCode();
```
native 关键词表明这是一个本地化方法，在本地库中用非 java 语言实现的，具体实现是通过把内存地址转换成整数，该方法是可以被重写的。

+ 在一个 java 程序中，同一个对象无论什么时候调用该方法都应该返回相同的整数值，前提是不修改该对象的 equals 方法。
+ 如果通过调用 equals 方法表明两个对象是相等的，那么这两个对象的 hashcode 就是相同的。
+ 两个对象通过 equals 判断不相等，但是这两个对象的 hashcode 是可以相等的，但是为了 Map 的效率，这种情况要尽量避免。


##### 3.3 equals
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
判断两个对象是否"相等",可重写该方法。Object 类中该方法的实现就是判断两个对象的引用是否相等。
源码中的注释说明了该方法满足如下四个特性：
+ 满足自反性，x.equals(x) 返回 true。
+ 满足对称性，x.equals(y) 和 y.equals(x) 返回的结果相等。
+ 满足传递性，x.equals(y) 和 y.equals(z) 则满足 x.equals(z)。
+ 满足一致性，在不修改 equals 方法的前提下，结果判断始终相同。

需要注意的是当重写此方法后，通常也需要重写 hashcode 方法，保持 equals 相同的对象也拥有相同的 hashcode。

##### 3.4 clone
```java
protected native Object clone() throws CloneNotSupportedException;
```
native 关键词表明这是一个本地化方法，在本地库中用非 java 语言实现的。
clone 的作用是创建并返回当前对象的一个拷贝,可以看到 clone 方法的定义上抛出了 CloneNotSupportedException 不支持拷贝的异常，这是因为Java认为并不是所有的类都需要 clone，只有实现了 Cloneable 接口的类才能调用 clone 方法，Cloneable 接口中并没有定义任何方法，实现它就是为了打一个标记。

这里会涉及到浅拷贝和深拷贝的概念。
###### 3.4.1 浅拷贝
拷贝的新对象，其成员变量只是做了赋值操作，基本数据类型的变量会开辟新的内存空间，而引用类型变量的引用指向还是和以前一样。当你修改引用成员变量后，新旧两个对象的该成员变量都发生了变化。
clone 方法默认是实现的浅拷贝。
###### 3.4.2 深拷贝
通过重写 clone 方法实现深拷贝，主要是为引用成员变量创建新的对象，这里需要注意的是当前类想要实现深拷贝，那么该类的继承链上的 clone 方法都要进行重写。
###### 3.4.3 工具
借助Apache Commons可以轻松的实现浅拷贝和深拷贝：
+ 深拷贝: SerializationUtils
+ 浅拷贝: BeanUtils

##### 3.5 toString
```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```
返回当前对象的文本描述，该文本描述应该准确、丰富、通俗易懂，推荐所有的子类都应该重写该方法。
Object类的toString方法返回的是类名加上十六进制的hashcode。
##### 3.6 notify
```java
public final native void notify();
```
notify 是本地化实现的方法，并且不能被重写。
notify 方法用来唤醒一个正在等待获取该对象 monitor 的线程。Java 的同步是通过 monitor 机制来实现的，在 Java 中每一个对象都会配一个 monitor，monitor 结构中定义了一个互斥标记，一个 entrySet 以及一个 waitSet，互斥标记用来标记当前是哪个线程占有了 monitor。线程访问到 syncronized 定义的临界区时会先进入 monitor 的 entrySet，通过判断互斥标记来争夺 monitor，当拥有 monitor 的线程执行 wait 方法后会进入 waitSet，等待下一个拥有 monitor 的线程来 notify 唤醒。
notify 方法的使用需要注意以下的情况:
+ 只有当前线程拥有 monitor 时才能调用 notify 方法唤醒在 waitSet 中的线程，否则会抛出 IllegalMonitorStateException 异常。
+ 当 waitSet 中有多个现在在等待时，唤醒的线程是随机选取的。
+ 被唤醒的线程和 entrySet 中等待的线程拥有相同的竞争 monitor 的权限。

##### 3.7 notifyAll 
```java
public final native void notifyAll();
```
和 notify 方法类似，区别在于会唤醒所有在 waitSet 中等待的线程。

##### 3.8 wait
wait 是本地化实现的方法，并且不能被重写。
在 Object 类中实现了三个不同参数的 wait 方法。
调用该方法的线程会进入 wait 状态，直到其他线程调用同一个对象的 notify() 或 notifyAll() 方法唤醒该线程，或者其他线程中断了该线程，也可以是通过设置等待的超时时间，让该线程在等待一段时间之后自动被唤醒。

只有当前线程拥有 monitor 时，才能调用该方法。
该方法会导致当前线程进入 monitor 结构中的 waitSet，并在之后退出同步语句块以及让出该对象的 monitor。进入 waitSet 的线程是不可用的状态，直到出现以下几种情况之一：
1. 其他线程调用了同一对象的 notify() 方法，并且随机选择了该线程进行唤醒。
2. 其他线程调用了同一对象的 notifyAll() 方法。
3. 其他线程通过 interrupt() 方法中断了该线程。
4. 通过设置等待的超时时间，让该线程在等待一段时间之后自动被唤醒。
当该线程再次获取同一对象的 monitor 时，同步语句块的执行状态恢复至和执行 wait 操作时一样。

wait() 必须放在循环中调用。
对于从 wait 中被 notify 的进程来说，它在被 notify 之后还需要重新检查是否符合执行条件，如果不符合，就必须再次被 wait，如果符合才能往下执行。举例说明，如果有两个生产者 A 和 B，一个消费者 C。当存储空间满了之后，生产者 A 和 B 都被 wait，进入等待唤醒队列。当消费者 C 取走了一个数据后，如果调用了 notify()，则 A 和 B 中的一个将被唤醒，假设 A 被唤醒，则 A 向存储空间放入了一个数据，至此空间就满了。A 执行了 notify() 之后，如果唤醒了 B，那么 B 不会再次判断是否符合执行条件，将直接执行 wait() 之后的程序，这样就导致向已经满了数据存储区中再次放入数据。错误产生。 

如果该线程之前被其他线程执行了 interrupt() 操作，当调用 wait() 时，会抛出 InterruptedException 异常，异常抛出后，该线程的中断状态会清除。

调用 wait() 的线程只会释放对应对象的 monitor，该线程持有的其他对象 monitor 不受影响。


###### 3.8.1 wait(long timeout)
```java
public final native void wait(long timeout) throws InterruptedException;
```
参数 timeout 设置了线程在 waitSet 中的等待时间。


###### 3.8.2 wait(long timeout, int nanos)
```java
public final void wait(long timeout, int nanos) throws InterruptedException {
   if (timeout < 0) {
      throw new IllegalArgumentException("timeout value is negative");
   }
   if (nanos < 0 || nanos > 999999) {
      throw new IllegalArgumentException("nanosecond timeout value out of range");
   }
   if (nanos > 0) {
      timeout++;
   }
   wait(timeout);
}
```
与 wait(long timeout) 方法类似。
JDK 中的方法描述说是提供了纳秒级别的等待时间控制（通过第二个参数控制），但是具体的代码实现还是毫秒级别的等待。只要第二个参数的纳秒数大于 0，并且在 1 毫秒的范围内，等待的毫秒数就直接加 1。真是简单又粗暴的实现，不清楚为什么要提供这个方法啊。

###### 3.8.3 wait()
```java
public final void wait() throws InterruptedException {
    wait(0);
}
```
与 wait(long timeout) 方法类似。该方法的内部实现调用了 wait(0)，即等待时间为无限。

##### 3.9 finalize
```java
protected void finalize() throws Throwable { }
```
方法体为空。

finalize() 是 Object 中的方法，当垃圾回收器将要回收对象所占内存之前被调用，即当一个对象被虚拟机宣告死亡时会先调用它 finalize() 方法，让此对象处理它生前的最后事情（这个对象可以趁这个时机挣脱死亡的命运）

在 Java 9 中，甚至明确将它标记为 deprecated！如果没有特别的原因，不要实现 finalize 方法，也不要指望利用它来进行资源回收。为什么呢？简单说，你无法保证 finalize 什么时候执行，执行的是否符合预期。相反使用不当会影响性能，导致程序死锁、挂起等。

实践中，由于 finalize 拖慢垃圾收集，导致大量对象堆积，也是一种典型的导致 OutOfMemeryError 的原因。Java 平台从Java 9 开始正在逐步使用 java.lang.ref.Cleaner 来替换掉原有的 finalize 实现。Cleaner 的实现利用了幻象引用（PhantomReference），这是一种常见的所谓 post-mortem 清理机制。

**资源用完即显式释放，或者利用资源池来尽量重用。**