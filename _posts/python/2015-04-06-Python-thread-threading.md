---
layout: post
title: "Python多线程编程(2):thread&amp;threading模块"
description: "Python多线程编程(2):thread&amp;threading模块"
category: "多线程"
tags: [多线程]
---
{% include JB/setup %}

<h2>thread模块</h2>

<p>提供的主要函数如下：</p>

<p>1.<code>start_new_thread(function,args,kwargs=None)</code> :第一个参数是线程函数,第二个参数是传递给线程函数的参数，它必须是tuple类型，kwargs是可选参数。</p>

<p>2.<code>exit_thread()</code>: 线程的结束可以等待线程自然结束，也可以在线程函数中调用thread.exit()或thread.exit_thread()方法。</p>

<p>3.<code>allocate_lock()</code>: 分配一个LockType类型的锁对象（注意，此时还没有获得锁）</p>

<p>4.<code>exit()</code>：退出线程</p>

<p><strong>LockType类型锁对象的函数</strong></p>

<p>1.<code>acquire(wait=None)</code>:尝试获取锁对象</p>

<p>2.<code>locked()</code>:如果获得了锁对象返回True，否则返回False</p>

<p>3.<code>release()</code>:释放锁</p>

<p>实例：</p>

<pre><code># coding=utf-8
import thread
import time

def func(flag):
    con = 1
    while con &lt; 4:
        print '%s run time:%s\n' % (flag, time.time())
        time.sleep(1)
        con += 1
    thread.exit_thread()

def run__():
    thread.start_new_thread(func, ('foo',))
    thread.start_new_thread(func, ('bar',))

if __name__ == '__main__':
    run__()
    time.sleep(2)       # 必须sleep
</code></pre>

<p>这种方法的缺点就在主线程要睡眠一段时间等待子线程全部结束，不然如果结束主线程，那么子线程都会结束。但是子线程会运行多长时间一般是很难确定的。</p>

<p>可以通过线程锁来觉得主线程是否可以退出了，用来代替子线程运行时间的不确定性。</p>

<p>主线程通过检测各个子线程释放锁的状态来确定是否需要关闭。</p>

<pre><code># coding=utf-8
__author__ = 'fang'
import time
import threading
import thread

#采用锁的方式来控制主线程退出

#记录和释放锁
def loop(lockObj):
    print 'lock:%s come in at:%s' %(lockObj, time.ctime())
    time.sleep(1)
    lockObj.release()       # 释放锁
    print 'lock:%s release' % lockObj

#线程控制器
def threadControl():
    locks = []      # 锁列表
    #生成锁对象
    for i in range(3):
        t = thread.allocate_lock()  #分配一个LockType类型的锁对象（注意，此时还没有获得锁）
        t.acquire()                 #尝试获取锁对象
        locks.append(t)

    #根据生成的锁对象启动线程
    for i in locks:
        thread.start_new_thread(loop, (i, ))

    #主线程检查每一个子线程的加锁状态
    for i in locks:
        while i.locked():       #如果获得了锁对象返回True，否则返回False
            pass

    print 'all done:', time.ctime()
</code></pre>

<p>thread模块用于处理底层的线程结构，对于线程的控制性不好，特别是主线程，在开发阶段最好使用更高度封装的<code>threading</code>.</p>

<h2>threading</h2>

<p><strong>threading模块对象:</strong></p>

<p>1.<code>Thread</code>:一个线程的执行对象</p>

<p>2.<code>Lock</code>:锁对象</p>

<p>3.<code>RLock</code>:可重入锁对象，使单线程可以再次获得已经获得了的锁（递归锁定）</p>

<p>4.<code>Condition</code>:条件变量，让一个线程停下来，等待其它线程满足了某个条件</p>

<p>5.<code>Event</code>:事件对象，通用的条件变量</p>

<p>6.<code>Semaphore</code>:信号量</p>

<p>7.<code>BoundedSemaphore</code>: 与Semaphore类似，只是它不允许超过初始值</p>

<p>8.<code>Timer</code>: 与Thread相似，不过它要等待一段时间才开始运行</p>

<p><strong>Thread对象的函数:</strong></p>

<p>1.<code>start()</code>:线程开始执行</p>

<p>2.<code>run()</code>:线程功能函数</p>

<p>3.<code>join(timeout=None)</code>: 程序挂起，直到线程结束;如果给了timeout，则最多阻塞timeout秒</p>

<p>4.<code>getName()</code>: 返回线程名字</p>

<p>5.<code>setName(name)</code>: 设置线程名字</p>

<p>6.<code>isAlive()</code>: 这个线程是否还在执行</p>

<p>7.<code>isDaemon()</code>:返回线程的daemon标志</p>

<p>8.<code>setDaemon(daemonic)</code>: 设置线程的daemon属性，一定要在调用start()函数前调用</p>

<p>三种方法创建线程。</p>

<p><strong>Type1： 创建一个Thread实例，传递一个函数</strong></p>

<pre><code># coding=utf-8
__author__ = 'fang'
import time
import threading

def threadFunc(flag):
    print '%s come at %s' % (flag, time.ctime())
    time.sleep(1)
    print '%s end at %s' % (flag, time.ctime())

def main():
    threads = []        # 线程列表
    for i in range(3):
        t = threading.Thread(target=threadFunc, name='thread_%s' %i, args=(i, ))
        threads.append(t)

    for i in threads:
        i.start()

    for i in threads:
        i.join()

if __name__ == '__main__':
    main()
</code></pre>

<blockquote>
  <p>这种方式就省了管理一堆锁的功夫了，只要对每个线程调用join()函数，就会等到线程结束。</p>
  
  <p>另外，如果你的主线程除了等线程结束外，还有其它的事情要做（如处理或等待其它的客户请求），那就不用调用join()。一旦线程启动后，就会一直运行，直到线程的函数结束，退出为止。所以只有在你要等待线程结束的时候才要调用join()。</p>
</blockquote>

<p><strong>Type2:创建一个Thread的实例，传给它一个可调用的类对象</strong></p>

<p>同上，只需把线程调用函数包装到类里即可，原先的threadFunc可进化如下，其他的同上：</p>

<pre><code>class threadFunc(object):
    def __init__(self, func, args, name=''):
        self.name = name
        self.func = func
        self.args = args

    def __call__(self, *args, **kwargs):
        self.res = self.func(*self.args)
        #如果python版本是1.6以下则使用如下
        #apply(self.func, self.args)
</code></pre>

<p>其实也就是使用python 魔法方法<code>__call__()</code>.</p>

<p><strong>Type3:从Thread类派生</strong></p>

<pre><code>class MyThread(threading.Thread):  
    def __init__(self, func, args, name=""):  
        threading.Thread.__init__(self)  
        self.name = name  
        self.func = func  
        self.args =args  

    def run(self):  
        self.res = self.func(*self.args)  
</code></pre>

<p>后面的代码同上，创建线程实例就是通过Thread派生出来的子类进行创建。</p>

<pre><code>t = MyThread(loop, (i, loops[i]), loop.__name__)  
</code></pre>

<hr/>

<p>参考:<a href="http://blog.csdn.net/whoami021/article/details/21265263">http://blog.csdn.net/whoami021/article/details/21265263</a></p>
