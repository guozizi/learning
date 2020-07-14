## python中的垃圾回收机制
python语言默认采用的垃圾收集机制是【引用计数法】

原理：每个对象维护一个ob_ref字段，用来记录该对象当前被引用的次数，每当新的引用指向该对象时，它的引用计数ob_ref加1，每当该对象的引用失效时计数ob_ref减1，一旦对象的引用计数为0，该对象立即被回收，对象占用的内存空间将被释放

缺点：它不能解决对象的“循环引用”（内存空间在使用完毕后未释放，造成内存泄漏）
      需要额外的空间维护引用计数  <br>

为了解决对象循环引用问题，python引入标记清除、分代回收两种GC机制
1、标记清除： <br>
引用计数的变量ref_count、gc_ref  <br>
它分为两个阶段：第一阶段是标记阶段，GC会把所有的『活动对象』打上标记，第二阶段是把那些没有标记的对象『非活动对象』进行回收  <br>

2、分代回收：是一种以空间换时间的操作方式，Python将内存根据对象的存活时间划分为不同的集合，每个集合称为一个代，Python将内存分为了3“代”，分别为年轻代（第0代）、中年代（第1代）、老年代（第2代），他们对应的是3个链表，它们的垃圾收集频率与对象的存活时间的增大而减小

 <https://jin-yang.github.io/post/python-garbage-collection.html>

 # python内存泄露
 1、对象被另一个生命周期特别长的对象所引用  <br>
 比如网络服务器，可能存在一个全局的单例ConnectionManager，管理所有的连接Connection，如果当Connection理论上不再被使用的时候，没有从ConnectionManager中删除，那么就造成了内存泄露  <br>
 2、循环引用中的对象定义了__del__函数  <br>
 如果定义了__del__函数，那么在循环引用中Python解释器无法判断析构对象的顺序，因此就不错处理

 <https://www.cnblogs.com/xybaby/p/7491656.html>

 ## python装饰器
 装饰器的作用就是为已经存在的对象添加额外的功能
 简单装饰器

    def use_logging(func):
        def wrapper(*args, **kwargs):
            logging.warn("%s is running" % func.__name__)
            return func(*args, **kwargs)
        return wrapper

    @use_logging #@符号是装饰器的语法糖，在定义函数的时候使用，避免再一次赋值操作
    def bar():
        print("i am bar")

    bar()

 带参数装饰器：实际上是对原有装饰器的一个函数封装，并返回一个装饰器。我们可以将它理解为一个含有参数的闭包

 类装饰器：类装饰器，相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点，使用类装饰器还可以依靠类内部的__call__方法

    class Foo(object):
        def __init__(self, func):
            self._func = func
        def __call__(self):
            print ('class decorator runing')
            self._func()
            print ('class decorator ending')

    @Foo
    def bar():
        print ('bar')

    bar()

 functools.wraps:使用装饰器极大地复用了代码，它能把原函数的元信息拷贝到装饰器函数中，这使得装饰器函数也有和原函数一样的元信息了

    from functools import wraps
    def logged(func):
        @wraps(func)
        def with_logging(*args, **kwargs):
            print (func.__name__ + " was called")
            return func(*args, **kwargs)
        return with_logging

    @logged
    def f(x):
       """does some math"""
       return x + x * x

    print (f.__name__)  # prints 'f'
    print (f.__doc__)   # prints 'does some math'

## 深拷贝浅拷贝
赋值引用(b=a): a 和 b 都指向同一个对象 <br>
浅拷贝(a.copy()): a 和 b 是一个独立的对象，但他们的子对象还是指向统一对象（是引用）  <br>
深度拷贝(copy.deepcopy(a)): a 和 b 完全拷贝了父对象及其子对象，两者是完全独立的  <br>
<https://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html>

## select、poll、epoll(io多路复用)
 <https://www.jianshu.com/p/dfd940e7fca2>
 
 
## 协程
 协程是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。  <br>
 
 与线程相比，协程更轻量。一个Python线程大概占用8M内存，而一个协程只占用1KB不到内存。协程更适用于IO密集型的应用。
 
 send
 
    def func():
        while True:
            print("before yield")
            x = yield
            print("after yield:",x)

    g = func()
    next(g) # 程序运行到yield并停在该处,等待下一个next
    g.send(1) # 给yield发送值1,这个值被赋值给了x，并且打印出来,然后继续下一次循环停在yield处
    g.send(2) # 给yield发送值2,这个值被赋值给了x，并且打印出来,然后继续下一次循环停在yield处
    next(g) # 没有给x赋值，执行print语句，打印出None,继续循环停在yield处
    
    # 输出
    before yield
    after yield: 1
    before yield
    after yield: 2
    before yield
    after yield: None
    before yield


<https://www.cnblogs.com/zingp/p/8678109.html> 
<https://www.cnblogs.com/fengf233/p/11548769.html>
<https://www.pythonf.cn/read/99110>

## 如何排查线上程序CPU或者内存占用过高的问题?
CPU占用过高的情况可能有两种：
- 任务本来就是CPU密集型，这是正常的情况
- 代码中出现了死循环之类的问题，属于代码本身的问题

我们可以通过top定位到占用CPU最高的进程，这里比如是1234，然后再通过top -H -p pid 或者ps -mp pid -o THREAD,tid,time，来定位究竟是哪个线程占用了过多的CPU  <br>
然后可以用pstack,strace和gdb,jstack这类工具来定位到具体哪个线程出了问题。我们也可以在启动程序的时候，记录线程号，然后根据线程号和对应的线程名，就能查找到哪个线程的问题了

存占用过高的情况可能是:
- 需要处理的文本很大，程序未做较多的优化
- 内存泄露

引起Python程序内存泄露的原因有两个:
- 对象被另一个生命周期特别长的对象所引用
- 循环引用中的对象定义了del函数，这个时候Python垃圾回收器并不知道该先回收哪个对象

我们可以使用gc模块和objgraph模块来解决内存泄露问题，objgraph模块可以将对象的引用关系可视化。并且在出现循环引用的时候，如果开启了Python的自动垃圾回收功能，那么不可达对象就会被垃圾回收器清除。关于循环引用的解决方法还有一个就是，如果业务允许（比如不需要保证a.b，b.a引用目标存活，如缓存，事件回调等）那么可以使用弱引用

## python 线程安全
## 进程和线程的区别
## 线程的生命周期
# Python的多线程的理解，多线程会有什么缺点？
## 进程线程区别、进程间的通信模式
## 多线程或者多进程并发的情况下，如何解决死锁问题呢
## Python中高阶函数是什么
## Python的__new__和__init__有什么区别
## python 以__开头的系统函数有哪些
## Python中"is"和"=="的区别
## Python的functools和itertools有用过什么吗？Collections用到什么？
## import一个包时过程是怎么样的
## Top k问题