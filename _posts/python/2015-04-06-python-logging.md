---
layout: post
title: "python模块深入之logging(一)"
description: "python模块深入之logging(一)"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p>源码位置:<a href="https://hg.python.org/cpython/file/2.7/Lib/logging/__init__.py">Lib/logging/<strong>init</strong>.py</a>，在2.3版本中才出现。</p>

<p>这个模块定义函数和类对app和库执行一个灵活的日志系统事件(implement a flexible event logging system).</p>

<p>关键好处就是所有的python模块能够分享通过标准库提供的logging API，所以你的应用程序日志可以包括自己的消息从第三方(third-party)模块集成。</p>

<p>The basic classes defined by the module, together with their functions, are listed below:</p>

<p>1.logger:loggers公开接口，应用程序代码直接使用</p>

<p>2.handler:处理程序发送日志记录的输出</p>

<p>3.filter:过滤器提供更细粒度决定哪些日志记录输出</p>

<p>4.filter:格式化最终日志输出</p>

<!--more-->

<h2>Logger 对象</h2>

<p>Loggers有如下属性和方法，注意Loggers对象不是直接实例化，总是通过module级别函数<code>logging.getLogger(name)</code>,通过同一名字多重调用<code>getLogger()</code>将总是返回一个同一Logger对象的引用。</p>

<p><code>name</code>可能是一个期间分割(period-separated)的等级值，就像<code>foo.bar.baz</code>(尽管它也能 plain foo)，在等级值列表中更深层Loggers是children of loggers higher up in the list。比如给定一个名为foo的logger对象，名为foo.bar,foo.bar.baz和foo.bam都是foo的后代。logger命名等级与python包的等级相似，同一(如果你组织你的loggers在每个module(per-module)基础通过被推荐的<code>logging.getLogger(__name__)</code>结构，因为在python命名空间中一个module的<strong>name</strong>是它自身的名称)</p>

<pre><code>&gt;&gt;&gt; import logging
&gt;&gt;&gt; logging.getLogger(__name__)
&lt;logging.Logger object at 0x106934610&gt;
&gt;&gt;&gt; __name__
'__main__'
</code></pre>

<p>日志等级：</p>

<p>Level   Numeric value</p>

<p>CRITICAL    50</p>

<p>ERROR   40</p>

<p>WARNING 30</p>

<p>INFO    20</p>

<p>DEBUG   10</p>

<p>NOTSET  0</p>

<h2>logging模块中的常用函数：</h2>

<p>参考：<a href="http://blog.csdn.net/jgood/article/details/4340740"> Python模块学习 ---- logging 日志记录（一）</a></p>

<p>如下示例：</p>

<pre><code># coding=utf-8
__author__ = 'fang'
import logging
import sys
import os

LEVELS = {
    'debug':logging.DEBUG,
    'info':logging.INFO,
    'warning':logging.WARNING,
    'error':logging.ERROR,
    'critical':logging.CRITICAL
}
if len(sys.argv)&gt;1:
    print sys.argv
    lname = sys.argv[1]

"""
logging.basicConfig([**kwargs]):
为日志模块配置基本信息。kwargs 支持如下几个关键字参数：
filename ：日志文件的保存路径。如果配置了些参数，将自动创建一个FileHandler作为Handler；
filemode ：日志文件的打开模式。 默认值为'a'，表示日志消息以追加的形式添加到日志文件中。如果设为'w', 那么每次程序启动的时候都会创建一个新的日志文件；
format ：设置日志输出格式；
datefmt ：定义日期格式；
level ：设置日志的级别.对低于该级别的日志消息将被忽略；
stream ：设置特定的流用于初始化StreamHandler；
"""
level = LEVELS.get(lname)
filename = os.path.join(os.getcwd(), 'log.txt')
filemode = 'a'
format = '%(asctime)s-%(levelname)s:%(message)s'

#为日志模块配置基本信息
logging.basicConfig(filename=filename,level=level,filemode=filemode,format=format)
logging.info('info..')      # 默认使用根root

"""
logging.getLogger([name])
创建Logger对象。日志记录的工作主要由Logger对象来完成。在调用getLogger时要提供Logger的名称（注：多次使用相同名称来调用getLogger，返回的是同一个对象的引用。），Logger实例之间有层次关系，这些关系通过Logger名称来体现，如：
p = logging.getLogger("root")
c1 = logging.getLogger("root.c1")
c2 = logging.getLogger("root.c2")
例子中，p是父logger, c1,c2分别是p的子logger。c1, c2将继承p的设置。如果省略了name参数, getLogger将返回日志对象层次关系中的根Logger。
"""
#创建Logger对象
log = logging.getLogger('root.test')

log.setLevel(logging.WARN) # 设置日志的级别。对于低于该级别的日志消息将被忽略,日志记录级别为WARNNING

"""
Logger.xxx(msg [ ,*args [, **kwargs]])
记录xxx级别的日志信息。参数msg是信息的格式，args与kwargs分别是格式参数。
"""
log.info('info') # 不会被记录
log.error('error')
log.warning('%s, %s, %s', *('warning', 'warning', '.....x'))

"""
Logger.log(lvl, msg[ , *args[ , **kwargs] ] )
记录日志，参数lvl用户设置日志信息的级别。参数msg, *args, **kwargs的含义与Logger.debug一样。
"""
logging.log(logging.DEBUG, 'logDebug')

"""
Logger.exception(msg[, *args])
以ERROR级别记录日志消息，异常跟踪信息将被自动添加到日志消息里。Logger.exception通过用在异常处理块中
"""
try:
    raise Exception, 'this is a exception'
except:
    log.exception('exception')
</code></pre>
