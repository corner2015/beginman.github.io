---
layout: post
title: "Python Sys._getframe"
description: "Python Sys._getframe"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p><code>sys._getframe([depth])</code></p>

<p>从调用堆栈返回一个框架对象。如果可选整数深度给定，则返回该帧的对象。如果比所述调用栈更深，ValueError被挂起。默认
对于深度是零，在调用栈顶部的框架返回。</p>

<pre><code>import sys
def testSys():
    print sys._getframe().f_code.co_filename            #当前文件路径：/Users/fang/project/demo/dic.py,也可以通过__file__获取
    print sys._getframe(0).f_code.co_name               #当前函数名：testSys
    print sys._getframe(1).f_code.co_name               #调用该函数的函数的名字，如果没有被调用，则返回&lt;module&gt;，貌似call stack的栈低
    print sys._getframe().f_lineno                      #当前行号 13
    print sys._getframe(0).f_code.co_filename           #当前文件路径
    print sys._getframe(0).f_back.f_code.co_filename    #当前文件路径


if __name__ == '__main__':
    testSys()
</code></pre>
