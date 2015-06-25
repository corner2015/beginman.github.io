---
layout: post
title: "python装饰器小结"
description: "python装饰器小结"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<h2>1.以函数对象作为参数</h2>

<p>先看两个例子</p>

<p><strong>代码1：</strong></p>

<pre><code>def deco(fn):
    print "bef..."
    fn()
    print "end..."
    return fn           # 返回原函数对象

def func():
    print 'fun'

myfun = deco(func)     # 将原函数对象func赋值给变量myfun
myfun()
myfun()
</code></pre>

<p>运行后的结果如下：</p>

<pre><code>bef...
fun
end...
fun
fun
</code></pre>

<p>执行过程如下：</p>

<p><img src="http://beginman.qiniudn.com/decorator1.png" alt="" /></p>

<p><strong>代码2：</strong></p>

<pre><code>def deco(fn):
    def wrapper():
        print "bef..."
        fn()
        print "end..."
                          # 这里不再返回原函数对象
    return wrapper           # 返回wrapper函数对象

def func():
    print 'fun'

myfun = deco(func)          # 将wrapper函数对象赋值给myfun
myfun()                     # 执行wrapper函数
myfun()                     # 执行wrapper函数
</code></pre>

<p>运行后的结果：</p>

<pre><code>bef...
fun
end... 
bef...
fun
end...
</code></pre>

<p>执行过程如下：</p>

<p><img src="http://beginman.qiniudn.com/decorator2.png" alt="" /></p>

<p>两种代码不同之处在于：</p>

<ul>
<li>代码1.发现新函数只在第一次被调用，且原函数多调用了一次</li>
<li>代码2.装饰函数返回内嵌包装函数对象保证了每次新函数都被调用</li>
</ul>

<!--more-->

<h2>2.装饰器</h2>

<p>如果将 <code>myfun = deco(func)</code>变相以下，就成了如下方式：</p>

<pre><code>@deco
def func():
    .....
</code></pre>

<p><strong>这种变相的方式就称为<code>装饰器</code>，所谓的装饰器就是<code>@</code>语法糖- Syntactic Sugar的形式来包装一个函数。这种包装作用就是在函数执行前处理某某，在函数执行后处理某某。</strong></p>

<p>如下实例：</p>

<pre><code>def deco(fn):
    def wrapper(*args, **kwargs):
        return fn(*args, **kwargs)

    return wrapper

def fun(a,b):
    return a+b

#将函数作为参数的直接形式
myfun = deco(fun)           # 将wrapper函数对象赋值给myfun
print myfun(100,200)        # 调用wrapper函数,返回fn回调函数的值

#装饰器形式
@deco
def foo(a, b):
    return a+b

print foo(100,200)          # 300
</code></pre>

<p>同时可以让装饰器带参数：</p>

<pre><code>def deco(*args, **kwargs):
    #内嵌一个真正的装饰函数
    def true_deco(fn):
        def wrapper(*args, **kwargs):
            #返回原函数执行的结果
            return fn(*args, **kwargs)

        # 返回wrapper函数装饰对象
        return wrapper

    print 'the deco values is:', args, kwargs
    # 返回真正的装饰函数 true_deco
    return true_deco


@deco(1,2)
def fun(a, b):
    return a + b

print fun(100, 200)
</code></pre>

<p>执行结果如下：</p>

<pre><code>the deco values is: (1, 2) {}
300
</code></pre>

<p><strong>总结：无论包装函数包裹了多少层，一定要返回一个真正的包装函数</strong>，</p>

<p>如下是多个包装函数的例子：</p>

<pre><code>def deco1(*args, **kwargs):
    print 'deco1 args:', args, kwargs
    def deco1__(fn):
        print u'1等包装函数'
        def wrapper(*args, **kwargs):
            print u'内嵌包装函数'
            return fn(*args, **kwargs)

        return wrapper
    return deco1__

def deco2(fn):
    print u'包装函数2'
    def wrapper(*args, **kwargs):
        return fn(*args, **kwargs)

    return wrapper


@deco1(1,2)
def foo(a, b):
    return a + b

print foo(100,200)

print '\n'+'*'*10+'\n'

@deco2
def foo2(a, b):
    return a + b

print foo2(100,200)

print '\n'+'*'*10+'\n'

@deco1(1,2)
@deco2
def foo3(a, b):
    return a + b

print foo3(100,200)

print '\n'+'*'*10+'\n'

myfoo = deco1(1,2)(deco2(foo))
print myfoo(100, 200)
</code></pre>

<p>输出结果如下：</p>

<pre><code>    deco1 args: (1, 2) {}
1等包装函数
内嵌包装函数
300

**********

包装函数2
300

**********

deco1 args: (1, 2) {}
包装函数2
1等包装函数
内嵌包装函数
300

**********

deco1 args: (1, 2) {}
包装函数2
1等包装函数
内嵌包装函数
内嵌包装函数
300
</code></pre>

<h2>3.类装饰器</h2>

<p><strong>类装饰器实现方式1：</strong></p>

<pre><code>class ClassDeco(object):
    def __init__(self, fn, *args, **kwargs):
        self.fn = fn
        self.arg = args
        self.kwargs = kwargs

    def __call__(self, *args, **kwargs):
        return self.fn(*args, **kwargs)


@ClassDeco
def fuc(a,b):
    return a + b

print fuc(1, 2)         # 3

myfunc = ClassDeco(fuc)
print myfunc(1, 2)      # 3
</code></pre>

<p><strong>类装饰器实现方式2：</strong></p>

<pre><code>class ClassDeco(object):
    def __init__(self, *args, **kwargs):
        self.arg = args
        self.kwargs = kwargs

    def __call__(self, fn, *args, **kwargs):
        def wrapper(*args, **kwargs):
            return fn(*args, **kwargs)

        return wrapper

@ClassDeco(100,200)
def fuc(a,b):
    return a + b

print fuc(1, 2)         # 3

myfunc = ClassDeco(100,200)(fuc)
print myfunc(1, 2)      # 3
</code></pre>

<p>如上实现了2个类装饰器，实现要点如下：</p>

<p><strong>如果类装饰器不需要参数</strong></p>

<ol>
<li><code>__init__</code> 定义被装饰的函数</li>
<li><code>__call__</code> 成为装饰函数，调用<strong>init</strong>中被装饰的函数</li>
</ol>

<p><strong>如果类装饰器需要参数</strong></p>

<ol>
<li><code>__init__</code> 初始化参数，<strong>不再</strong>定义被装饰的函数</li>
<li><code>__call__</code> 成为装饰函数，同时被装饰的函数作为其参数传入</li>
</ol>

<h2>3.装饰器带来的副作用</h2>

<p>副作用就是：<strong>被装饰的函数转换成了另一个函数</strong></p>

<pre><code>def deco(fn):
    def wrapper():
        print fn.__name__       # foo
    return wrapper

@deco
def func():
    print 'hello'

func()

print func.__name__     # wrapper
</code></pre>

<p>这里就要用到<code>warps</code>装饰器了，<strong>它可以把被封装函数的</strong><strong>name</strong>、module、<strong>doc</strong>和 <strong>dict</strong>都复制到封装函数去(模块级别常量WRAPPER_ASSIGNMENTS, WRAPPER_UPDATES)</p>

<pre><code>from functools import wraps
    def deco(fn):
        @wraps(fn)
        def wrapper():
            print fn.__name__   # func
        return wrapper

@deco
def func():
    print 'hello'

func()

print func.__name__     # func
</code></pre>

<p>这里装饰器就告一段落了，回头整理开发过程中经典的装饰器例子。</p>
