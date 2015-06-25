---
layout: post
title: "Python多线程编程(1)：基础"
description: "Python多线程编程(1)：基础"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<h2>一.理解线程与进程</h2>

<h3>1.1 进程(process)</h3>

<p>进程是计算机中已运行程序的实体，是程序的基本执行实体，是线程的容器。它有两种运行方式：同步（循序）和异步（平行）。</p>

<p><img src="http://www.bbc.co.uk/schools/gcsebitesize/design/images/fd_production_process.gif" alt="" /></p>

<h3>1.2 进程与程序的关系</h3>

<p>程序本身只是指令、数据及其组织形式的描述，进程才是程序（那些指令和数据）的真正运行实例。若干进程有可能与同一个程序相关系，计算机以进程的形式加载多个程序到存储器中。</p>

<p>以下参考可能会更加清晰了解程序与进程的关系：</p>

<blockquote>
  <p>程序只是一个普通文件，是一个机器代码指令和数据的集合，这些指令和数据存储在磁盘上的一个可执行映象（Executable Image）中，所以，程序是一个静态的实体。这里，对可执行映象做进一步解释，可执行映象就是一个可执行文件的内容，例如，你编写了一个C源程序，最终这个源程序要经过编译、连接成为一个可执行文件后才能运行。源程序中你要定义许多变量，在可执行文件中，这些变量就组成了数据段的一部分；源程序中的许多语句，例如“ i++; for(i=0; i&lt;10; i++);”等，在可执行文件中，它们对应着许多不同的机器代码指令，这些机器代码指令经CPU执行，就完成了你所期望的工作。可以这么说：程序代表你期望完成某工作的计划和步骤，它还浮在纸面上，等待具体实现。而具体的实现过程就是由进程来完成的，进程可以认为是运行中的程序，它除了包含程序中的所有内容外，还包含一些额外的数据。程序和进程的组成如图所示。</p>
</blockquote>

<p><img src="http://oss.org.cn/kernel-book/ch04/4.1.files/image002.gif" alt="" /></p>

<p><!--more--></p>

<h3>1.3 进程与线程的关系</h3>

<p>下图可见其关系</p>

<p><img src="http://www.herbaat.com/wp-content/uploads/2014/01/process-and-thread.png" alt="" /></p>

<h3>1.4 线程(thread)</h3>

<p><strong>是操作系统能够进行运算调度的最小单位</strong>。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。</p>

<p>线程有四种状态：</p>

<p>产生（spawn）</p>

<p>中断（block）</p>

<p>非中断（unblock）</p>

<p>退出（finish）</p>

<h3>1.5  多线程</h3>

<p>线程是程序中一个单一的顺序控制流程.在单个程序中同时运行多个线程完成不同的工作,称为多线程。</p>

<p>为什么要引入多线程呢？有以下几个<strong>优点</strong>：</p>

<blockquote>
  <p>1.改进性能和并发(improved performance and concurrency),对于某些应用程序,可以提高性能和并发性通过使用多线程。2.减少了所需的服务器数量。</p>
</blockquote>

<p><strong>缺点</strong>如下：</p>

<blockquote>
  <p>1.调试困难 .  2.并发难以管理，如果有大量的线程,会影响性能,因为操作系统需要在它们之间切换且更多的线程需要更多的内存空间。</p>
</blockquote>

<p>更多内容，参考<a href="http://developer.51cto.com/art/201306/398002.htm"><strong>多线程的优点和代价</strong></a></p>

<h3>1.6  线程与进程的关系</h3>

<p>在<a href="http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html"><strong>进程与线程的一个简单解释</strong></a> 这篇博客中，能显而易见地弄清楚它们之间的联系。</p>

<p>更多内容，可以参考<a href="http://www.qnx.com/developers/docs/6.4.1/neutrino/getting_started/s1_procs.html?lang=cn#Threads_and_processes"><strong>Processes and Threads</strong></a></p>

<h2>二.Python多线程</h2>

<h3>2.1  python解释器</h3>

<p>python解释器将源代码转换为字节码然后执行的过程，这里的解释执行是相对于编译执行而言。更多内容参考<a href="http://www.wangyuxiong.com/archives/51258"><strong>Python解释执行原理</strong></a></p>

<h3>2.2  python虚拟机</h3>

<p>Python代码执行是由python虚拟机控制，在python虚拟机中同时只有一个线程执行，相当于单CPU运行多个进程，但是任意时刻只用一个进程在CPU中运行。</p>

<h3>2.3  python全局解释器锁</h3>

<p>对python虚拟机访问是通过python全局解释器锁(global interpreter lock , GIL)控制，实现相当于一把锁，在进程与线程概念中，存在着共享内存，其他线程必须等它结束，才能使用这一块内存。GIL就是锁住然后打开的不断循环的过程，用以防止多个线程同时读写某一块内存区域。</p>

<p><img src="http://deliveryimages.acm.org/10.1145/960000/959339/7124f1.png" alt="" /></p>

<h3>2.4  time.sleep()演示进程工作</h3>

<pre><code>#!/usr/bin/env python
# coding=utf-8
from time import sleep,ctime

def foo1():
    print 'foo1 starting..'
    sleep(4)
    print 'foo1 sleep 4 seconds!'

def foo2():
    print 'foo2 starting..'
    sleep(2)
    print 'foo2 sleep 2 seconds'

def main():
    foo1()
    foo2()

if __name__ == '__main__':
    main()
</code></pre>

<p>在单线程中顺序执行后，执行时间为这两个函数的总和，大约6秒多。</p>

<h2>三.thread模块的使用</h2>

<p>在《Python核心编程》中，提示要<strong>避免使用thread模块，而使用更高级的threading.</strong>，一个重要原因就是当主进程（也叫父进程，表示创建了线程的进程）结束后，其所有线程都会被强制结束掉，无任何提醒，也不会用正常的清除工作。注意，在python3中，thread更名为_thread.</p>

<p>常用函数：</p>

<pre><code>thread.start_new_thread(function, args[, kwargs])
#创建一个新线程并返回它的标识，其方法类似apply(),需要一个函数名和参数（必须是元组类型的，若无则传`()`）

thread.exit()
# 退出线程，若没有捕获则触犯SystemExit异常。
</code></pre>

<p>更多thread常用函数，参考<a href="https://docs.python.org/2/library/thread.html?highlight=thread#thread"><strong>文档16.3. thread — Multiple threads of control</strong></a></p>

<p>接下来实现上面的程序：</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
# try thread.
import thread
from time import sleep

def foo1():
    print 'foo1 starting...'
    sleep(4)
    print 'foo1 sleep 3 seconds'

def foo2():
    print 'foo2 starting...'
    sleep(2)
    print 'foo2 sleep 4 seconds'

def main():
    thread.start_new_thread(foo1,()) #create a thread
    thread.start_new_thread(foo2,()) #create a thread
    sleep(6) #如果不加这个或者sleep时间小于（2+4）则也会出错

if __name__ == "__main__":
    main()
</code></pre>

<p>注意，如果不在main()函数后添加sleep(6)则会<strong>报错</strong>，Unhandled exception in thread started by sys.excepthook is missing lost sys.stderr。这就体现了<strong>当主进程（也叫父进程，表示创建了线程的进程）结束后，其所有线程都会被强制结束掉</strong>。添加在主函数添加sleep()的目的就是让主线程稍微停下等待所有的子线程结束后再继续执行代码。</p>

<p>理想状态下，通过多线程实现两个程序的并发执行，程序运行的总时间就是线程运行时间最慢的一个。这里就是4秒了。
但是为了管理线程，主线程不得不加sleep来等待所有子线程，这样一来运行时间不比单线程短，还有一点由于多线程运行时间不确定，所以单单用sleep控制主线程是不靠谱的，这里就要用到<strong>锁</strong>了。</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
import thread
from time import sleep

loops = [4,2]

def foo(nloop,nsec,lock):
    print '%s starting...' %nloop
    sleep(nsec)
    print '%s sleep %s second' %(nloop,nsec)
    lock.release() #释放锁

def main():
    locks= []  # lock object list
    nloops = range(len(loops))

    # create LockType
    for i in nloops:
        lock = thread.allocate_lock() # 分配LockType类型的锁对象
        lock.acquire() # 尝试获取锁对象
        locks.append(lock) 

    # create thread
    for i in nloops:
        thread.start_new_thread(foo,(i,loops[i],locks[i]))
        while locks[i].locked():pass  # 返回锁对象

if __name__ == '__main__':
    main()
</code></pre>

<h2>四.threading模块的使用</h2>

<p>这里介绍threading的Thread类，更多threading详尽文档<a href="https://docs.python.org/2/library/threading.html?highlight=threading#lock-objects"><strong>16.2. threading — Higher-level threading interface</strong></a></p>

<pre><code>class threading.Thread(group=None, target=None, name=None, args=(), kwargs={})
</code></pre>

<h3>4.1 Thread对象的参数解释：</h3>

<p>group 应为 None，这参数是为将来可能出现的 ThreadGroup 类准备的（现在还没有实现）</p>

<p>target 为将被 run() 方法调用的可调用对象。如果是 None，那就意味着什么也不做</p>

<p>name 是本线程的名字，默认会分配一个形如“Thread-N”的名字，其中 N 是一个十进制数</p>

<p>args 是给 target 准备的非关键字参数</p>

<p>kwargs 是给 target 准备的关键字参数</p>

<p>daemon 用来设定线程的 daemon 属性，如果使用默认值（None），将从当前进程中继承</p>

<p>如果 Thread 的子类重写了构造函数，那么一定要确保在子类构造函数的第一行先调用父类的构造函数（Thread.<strong>init</strong>()）</p>

<blockquote>
  <p>在创建了 Thread 对象后，通过调用它的 start() 方法，run() 方法的内容会在一个新线程里被执行。
  一旦一个线程开始执行，那么他就处于“alive”的状态，除非 run() 方法执行完毕，或引发了一个未处理的异常。Thread 对象的 is_alive() 方法可以用来检测对象当前的状态。线程A 可以调用线程B 的 join() 方法，调用后线程A 会被挂起，直到线程B 结束。每个线程都有自己的名字。可以在调用构造器时通过 name=None 来设定，或者实例化后通过 name 属性来访问.线程可以被标记为“daemon thread”，这表示主程序可以不管他们死活。。。我是说，当一个程序里只剩下“daemon thread”没有结束时，程序就会直接退出。这个标记可以通过对象的 setDaemon() 方法来设定，但一定要在调用 start() 之前。对象的 daemon 属性和 isDaemon() 方法都可以用来检查它的标记。Python 程序的初始线程叫做“main thread”，当然他肯定不是“daemon thread”（Daemon　可参考《Python核心编程２》守护线程，就是在进程退出时，不用等待这个线程退出。）</p>
</blockquote>

<h3>4.2　Thread 类的属性和方法：</h3>

<p><code>start()</code></p>

<p>开始线程的执行</p>

<p>本方法至多只能调用一次。它会在独立线程中执行 run() 方法</p>

<p>如果对一个对象多次调用本方法，会引发 RuntimeError 异常</p>

<p><code>run()</code></p>

<p>包含了线程实际执行的内容</p>

<p>本方法可以在子类中覆盖。否则默认调用传给构造器的 target 参数（和 args，kwargs 参数）</p>

<p>常用操作如下：</p>

<pre><code> def run(self):
        apply(self.func,self.args)　＃应用apply函数
</code></pre>

<p><code>join(timeout=None)</code></p>

<p>程序挂起，直到线程结束（正常结束，或引发未处理异常，或超出 timeout 的时间）。</p>

<p>timeout 参数是一个以秒为单位的浮点数。当给出 timeout 参数时，因为 join() 方法总是返回 None，你应该随即调用 is_alive() 方法，来判断子线程到底是终结了，还是超时了。</p>

<p>如果没有给出 timeout 参数，那么调用者的进程会一直阻塞直到本进程结束。</p>

<p>一个线程可以被 join() 多次</p>

<p>当调用 join() 方法可能引发死锁，或被调用者的进程还未 start() 时，都会引发一个 RuntimeError</p>

<p><code>getName() / setName()</code></p>

<p>前者用于返回线程名字，后者用于设置线程名字</p>

<p><code>is_alive()</code></p>

<p>返回本进程是否是 alive 状态</p>

<p><code>daemon</code></p>

<p>布尔值，标明本进程是否为“daemon thread”。一定要在 start() 被调用前设定，否则引发 RuntimeError。初始值继承自当前线程，因此从主线程创建的子线程默认都是 False，而从“daemon thread”创建的子线程默认都是 True
当只剩下“daemon thread”为 alive 状态时，整个程序就会退出</p>

<p><code>isDaemon() / setDaemon()</code></p>

<p>为后向兼容而保留的方法，现在请直接访问 daemon 属性</p>

<h3>4.3　执行方式</h3>

<p>以下用三种方式实现多线程，由可用性而言，推荐最后一种。</p>

<p><strong>方式一:创建Thread实例，传递一个函数</strong></p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
from time import sleep
import threading

def foo(sec):
    print '%s starting..' % sec
    sleep(sec)
    print '%s end..' %sec

threads = []
for i in range(5):
    t = threading.Thread(target=foo,args=(i,))  # 创建Thread实例并传参
    threads.append(t)

for i in threads:
    i.start()  # 启动所有
    i.join()    # 等待结束
</code></pre>

<p><strong>方式二：创建Thread实例，传递一个可调用的类对象</strong></p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
from time import sleep
import threading

    class Mythread(object):
        def __init__(self,func,args):
            self.func = func
            self.args = args

        def __call__(self):
            apply(self.func,self.args)

def foo(i):
    print '%s starting..' %i
    sleep(i)
    print 'sleep %s' %i

threads = []
for i in range(5):
    t = threading.Thread(target=Mythread(foo,(i,))) # 传递可调用类对象
    threads.append(t)

for i in threads:
    i.start()
    i.join()
</code></pre>

<p><strong>方式三：从Thread派生出一个子类，创建这个子类的实例</strong></p>

<pre><code>#!/usr/bin/env python
# coding=utf-8

from time import sleep
import threading

class Mythread(threading.Thread):
    def __init__(self,func,args):
        threading.Thread.__init__(self)
        self.func = func
        self.args = args

    def run(self):
        apply(self.func,self.args)

def foo(i):
    print '%s starting..' %i
    sleep(i)

threads = []
for i in range(5):
    t = Mythread(foo,(i,))
    threads.append(t)

for t in threads:
    t.start()
    t.join()
</code></pre>

<h3>4.4 　一个线程流程的简单演示</h3>

<p>线程有5种状态，状态转换的过程如下图所示：</p>

<p><img src="http://images.cnblogs.com/cnblogs_com/huxi/WindowsLiveWriter/Python_11F5/thread_stat_simple_3.png" alt="" /></p>

<p>图片来源<a href="http://www.cnblogs.com/huxi/archive/2010/06/26/1765808.html"><strong>Python线程指南</strong></a></p>

<p>下面是一个简单的例子</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
import threading
from time import sleep

class Mythead(threading.Thread):
    def run(self):
        for i in range(3):
            sleep(1) # 挂起1s
            ＃self.name是该线程的名字，默认会分配一个形如“Thread-N”的名字，其中 N 是一个十进制数
            print u'线程:%s,索引%s' % (self.name,i)　　　

if __name__ == '__main__':
    for i in range(3):
        t = Mythead()
        t.start()
</code></pre>

<p>运行结果为：</p>

<pre><code>线程:Thread-1,索引0
线程:Thread-2,索引0
线程:Thread-3,索引0
线程:Thread-2,索引1
线程:Thread-3,索引1
线程:Thread-1,索引1
线程:Thread-2,索引2
线程:Thread-1,索引2
线程:Thread-3,索引2
</code></pre>

<p>从上面的运行结果来看，多线程程序的执行顺序是不确定的。当执行到sleep语句时，线程被阻塞（Blocked）（满足阻塞条件）直到sleep结束（阻塞结束）后，线程进入就绪（Runnable）状态，等待调度，执行完run()之后线程就会死亡。</p>

<h3>4.5 自定义线程名称</h3>

<p>从上面代码可以看出，线程名称默认会分配一个形如“Thread-N”的名字，其中 N 是一个十进制数。当然我们也可以自定义，如下：</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
import threading
from time import sleep
def foo():
    print threading.currentThread().getName(),'starting..'　　　＃currentThread()获取当前线程，getName()返回线程名称
    sleep(2)
    print threading.currentThread().getName(),'Exiting..'

f1 = threading.Thread(name='mythread', target=foo)                 # 设置线程名字为mythread.
f2 = threading.Thread(target=foo)

f1.start()
f2.start()
</code></pre>

<p>输出结果如下：（<strong>注意：结果是无序的，不确定的</strong>）</p>

<pre><code>mythread starting..
Thread-1 starting..
Thread-1 Exiting..
mythread Exiting..
</code></pre>

<p><strong><code>logging</code>打印输出：</strong></p>

<p>实际项目中往往不会让你print,而是利用logging,它能通过格式化字符串(<code>%(threadName)s</code>)支持线程日志的输出。下面是改写上面的程序：</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
import threading
from time import sleep
import logging
＃配置logging，以“模式名/线程名字(10个字符)/信息”方式输出。　
logging.basicConfig(
    level = logging.DEBUG,
    format = '[%(levelname)s](%(threadName)-10s) %(message)s'
    )
def foo():
    logging.debug('Starting...')
    sleep(2)
    logging.debug('Exiting...')

threading.Thread(name='mythread',target=foo).start()
threading.Thread(target=foo).start()
</code></pre>

<p>结果为：</p>

<pre><code>[DEBUG](mythread  ) Starting...
[DEBUG](Thread-1  ) Starting...
[DEBUG](mythread  ) Exiting...
[DEBUG](Thread-1  ) Exiting...
</code></pre>

<p>在改进一点就是写入到log日志里面去。</p>

<h3>4.6 设置守护线程</h3>

<p>这里要区分<code>守护进程</code>与<code>守护线程</code>的概念，</p>

<h2>五.参考</h2>

<p><a href="http://zh.wikipedia.org/wiki/%E8%BF%9B%E7%A8%8B">1.WIKI之进程</a></p>

<p><a href="http://oss.org.cn/kernel-book/ch04/4.1.htm">2.深入分析Linux内核源码之进程和程序（Process and Program）</a></p>

<p><a href="http://zh.wikipedia.org/wiki/%E7%BA%BF%E7%A8%8B">3.WIKI之线程</a></p>

<p><a href="http://my.oschina.net/lionets/blog/194577">4.Python的多线程编程模块 threading 参考</a></p>
