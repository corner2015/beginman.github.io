---
layout: post
title: "glob 模块"
description: "glob 模块"
category: "python模块"
tags: [python模块]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<h1>glob模块</h1>

<p>glob是Python标准库之一，主要作用就是依据Unix Shell规则找出对应位置与之匹配的文件名。主要方法就是<code>glob.glob(pattern)</code>。</p>

<p>这里的规则<code>pattern</code>与正则不同，仅仅有以下几个：</p>

<blockquote>
  <p>1.<code>*</code> 匹配0个或者一个</p>
</blockquote>

<pre><code>&gt;&gt;&gt; glob.glob('/home/beginman/learn/*.sh')  
# 匹配该目录下shell脚本
</code></pre>

<p>对于多级目录<code>glob</code>并不会递归搜索，可以用*进行手动递归</p>

<pre><code>&gt;&gt;&gt; glob.glob('/home/beginman/*/*/*.py')
</code></pre>

<blockquote>
  <p>2.<em>?</em> 匹配单个字符</p>
</blockquote>

<pre><code>&gt;&gt;&gt; glob.glob('/home/beginman/learn/test?.py')
# 输出 ：test1.py test2.py  (省略..)
</code></pre>

<blockquote>
  <p>3.<code>[]</code>区间，如[0-9],[a-z]等</p>
</blockquote>

<p><strong>应用：</strong></p>

<pre><code>[python]
#!/usr/bin/env python
# coding=utf-8
import itertools as it, glob, os

def chainFiles(*patterns):
    return it.chain.from_iterable(glob.glob(pattern) for pattern in patterns)

for file in chainFiles('*.sh', '*.txt'):
    print '%-15s' %file, os.path.realpath(file)

[/python]

# 输出：
tty.sh          /home/beginman/learn/tty.sh
1.sh            /home/beginman/learn/1.sh
newTest.txt     /home/beginman/learn/newTest.txt
test.txt        /home/beginman/learn/test.txt
newNumber.txt   /home/beginman/learn/newNumber.txt
1.txt           /home/beginman/learn/1.txt
</code></pre>
