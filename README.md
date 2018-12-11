# ThreadLocalVolatileTree
ThreadLocal，Volatile源代码技术研究


![](https://i.imgur.com/THhYN8u.png)

<pre>
      ThreadLocal的实现是这样的：每个Thread 维护一个 ThreadLocalMap 映射表，这个映射
    表的 key 是 ThreadLocal实例本身，value 是真正需要存储的 Object。

      也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 
    ThreadLocalMap 获取 value。值得注意的是图中的虚线，表示 ThreadLocalMap 是使
    用 ThreadLocal 的弱引用作为 Key 的，弱引用的对象在 GC 时会被回收。

   
    ThreadLocal为什么会内存泄漏：

        ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强
     引用来引用它，那么系统 GC 的时候，这个ThreadLocal势必会被回收，这样一来，
     ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry
     的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一
     条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法
     回收，造成内存泄漏。

     其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在
     ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key
     为null的value
</pre>

<pre>
为什么使用弱引用

      1）key 使用强引用：引用的ThreadLocal的对象被回收了，但是ThreadLocalMap还持
      有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
      2）key 使用弱引用：引用的ThreadLocal的对象被回收了，由于ThreadLocalMap持
      有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一
      次ThreadLocalMap调用set,get，remove的时候会被清除。

      比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没
      有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引
      用ThreadLocal不会内存泄漏，对应的value在下一次ThreadLocalMap调用set,get,
      remove的时候会被清除。

      因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，
      如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。
</pre>