---
layout: post
title: "python inspect模块"
description: "python inspect模块"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p><code>inspect</code>模块：自省：</p>

<p>关于自省，<a href="http://kb.cnblogs.com/page/87128/">参考这里</a><br />
这里列举主要的用法：</p>

<h3>1检查对象类型</h3>

<pre><code>is{module|class|function|method|builtin}(obj): 
</code></pre>

<p>检查对象是否为模块、类、函数、方法、内建函数或方法。</p>

<pre><code>isroutine(obj): 
</code></pre>

<p>用于检查对象是否为函数、方法、内建函数或方法等等可调用类型。用这个方法会比多个<code>is*()</code>更方便，不过它的实现仍然是用了多个<code>is*()</code>。</p>

<pre><code>im = cat.sayHi
if inspect.isroutine(im):
    im()
</code></pre>

<p>对于实现了<strong>call</strong>的类实例，这个方法会返回False。如果目的是只要可以直接调用就需要是True的话，不妨使用isinstance(obj, collections.Callable)这种形式。我也不知道为什么Callable会在collections模块中，抱歉！我大概是因为collections模块中包含了很多其他的ABC(Abstract Base Class)的缘故吧：）</p>

<!--more-->

<h3>2.获取对象信息</h3>

<pre><code>getmembers(object[, predicate]): 
</code></pre>

<p>这个方法是dir()的扩展版，它会将dir()找到的名字对应的属性一并返回，形如[(name, value), ...]。另外，predicate是一个方法的引用，如果指定，则应当接受value作为参数并返回一个布尔值，如果为False，相应的属性将不会返回。使用is*作为第二个参数可以过滤出指定类型的属性。</p>

<pre><code>getmodule(object): 
</code></pre>

<p>它返回object的定义所在的模块对象。</p>

<pre><code>get{file|sourcefile}(object): 
</code></pre>

<p>获取object的定义所在的模块的文件名|源代码文件名（如果没有则返回None）。用于内建的对象（内建模块、类、函数、方法）上时会抛出TypeError异常。</p>

<pre><code>get{source|sourcelines}(object): 
</code></pre>

<p>获取object的定义的源代码，以字符串|字符串列表返回。代码无法访问时会抛出IOError异常。只能用于module/class/function/method/code/frame/traceack对象。</p>

<pre><code>getargspec(func): 
</code></pre>

<p>仅用于方法，获取方法声明的参数，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 默认值元组)。如果没有值，将是空列表和3个None。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。</p>

<pre><code>def add(x, y=1, *z):
    return x + y + sum(z)
print inspect.getargspec(add)
#ArgSpec(args=['x', 'y'], varargs='z', keywords=None, defaults=(1,))
</code></pre>

<p>---s</p>

<pre><code>getargvalues(frame): 
</code></pre>

<p>仅用于栈帧，获取栈帧中保存的该次函数调用的参数值，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 帧的locals())。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。</p>

<pre><code>def add(x, y=1, *z):
    print inspect.getargvalues(inspect.currentframe())
    return x + y + sum(z)
add(2)
#ArgInfo(args=['x', 'y'], varargs='z', keywords=None,   locals={'y': 1, 'x': 2, 'z': ()})
</code></pre>

<p>--s</p>

<pre><code>getcallargs(func[, *args][, **kwds]): 
</code></pre>

<p>返回使用args和kwds调用该方法时各参数对应的值的字典。这个方法仅在2.7版本中才有。</p>

<pre><code>getmro(cls): 
</code></pre>

<p>返回一个类型元组，查找类属性时按照这个元组中的顺序。如果是新式类，与cls.<strong>mro</strong>结果一样。但旧式类没有<strong>mro</strong>这个属性，直接使用这个属性会报异常，所以这个方法还是有它的价值的。 
    print inspect.getmro(Cat)
    #(<class '__main__.Cat'>, <type 'object'>)
    print Cat.<strong>mro</strong>
    #(<class '__main__.Cat'>, <type 'object'>)
    class Dog: pass
    print inspect.getmro(Dog)
    #(<class __main__.Dog at 0x...>,)
    print Dog.<strong>mro</strong> # AttributeError</p>

<p>--s</p>

<pre><code>currentframe(): 
</code></pre>

<p>返回当前的栈帧对象。
其他的操作frame和traceback的函数请查阅inspect模块的文档，用的比较少，这里就不多介绍了。</p>

<h3>3实例：</h3>

<pre><code>&gt;&gt;&gt;import inspect
&gt;&gt;&gt;inspect.isbuiltin(abs)
True

&gt;&gt;&gt;import os
&gt;&gt;&gt;inspect.ismodule(os)
True

&gt;&gt;&gt;class C:pass
&gt;&gt;&gt;inspect.isclass(C)
True

&gt;&gt;&gt;class C(object):
    ...def foo(self): pass
    ...def bez(self): pass

&gt;&gt;&gt;inspect.getmembers(C, inspect.ismethod)
[('bez', "&lt;"unbound method C.bez"&gt;"), ('foo', "&lt;"unbound method C.foo"&gt;")]


&gt;&gt;&gt; inspect.getfile(os)
'/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/os.pyc'
&gt;&gt;&gt; inspect.getsourcefile(os)
'/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/os.py'
&gt;&gt;&gt; 

&gt;&gt;&gt; def foo(x, y=1, *z, **kwarg):
...     pass
... 
&gt;&gt;&gt; inspect.getargspec(foo)
ArgSpec(args=['x', 'y'], varargs='z', keywords='kwarg', defaults=(1,))
&gt;&gt;&gt;
</code></pre>
