##                                   JAVA引用类型

除了基本数据类型以外，其他的数据对象都是指向各个对象的引用，我们用  Object   a =new   Object( )   我们实际操控的是a ,这个a其实就像指针，像遥控器，我们不直接操作对象，我们操作对象的引用 

#### 强引用：

用new关键字new出来的对象就是强引用类型，强引用的对象很难被GC回收，虚拟机宁愿抛出OOM也不会调用GC清除强引用的对象。如果这个强引用超过了它的作用域，或者是被强制赋值为null，那么这个对象就可以被回收的了，被回收的时候还是要看具体的垃圾回收策略。

#### 软引用：

比强引用的生命周期短一些，只有当内存不足的时候才会去试图回收软引用的对象，JVM会在抛出OOM之前回收软引用对象。可以通过引用队列ReferenceQueue结合使用，如果一个软引用引用的对象被回收了，那么JVM会把这个软引用添加到与之相关的队列中，可以通过poll( )方法来检查这个引用队列里面的引用相关的对象是否被回收，如果队列为空，那么就返回null,否则返回队列中的第一个引用对象
软引用通常来实现内存敏感的缓存，空间足就保留，空间不足就清除

#### 弱引用：

弱引用的生命周期比软引用还要短，当垃圾回收区域扫描内存的时候，不管当前内存空间是否足够，都会清除软引用对象，也可以和引用队列一起使用，弱引用也是引用于内存紧张的缓存。

#### 虚引用：

也叫住做幻想引用，无法通过虚引用访问这个对象的任何属性或者函数，虚引用仅仅是提供了一个确保对象被finalize以后，做某些操作的机制，虚引用随时都可能会被GC回收，虚引用必须和引用队列一起使用，如果GC在回收一个对象之前发现这个对象还存在虚引用，那么就会把这个虚引用放到所对应的队列当中，那么就可以在所引用的对象被回收之前采取一些操作。可以用来跟踪对象被垃圾回收器回收的过程。