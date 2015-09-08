---
layout: post
title: "python计数与collections模块"
description: "python计数与collections模块"
category: "python"
tags: [python模块]
---
{% include JB/setup %}
<p>python 计算一个元素重复的次数可用多种方式：</p>

<pre><code># coding=utf-8

import collections
lis = ['a',1,2,3,4,1,2,3,'b','c','a','c','a',2]

def dict_count():
     #方式1
    dic = {}
    for i in lis:
        if str(i) in dic:
            dic[str(i)] += 1
        else:
            dic[str(i)] = 1

    return dic

def defaultdict_count():
     #方式2
    d = collections.defaultdict(int)
    for i in lis:
        d[i] += 1

    return d

def set_count():
     #方式3
    s = set(lis)
    dic = {}
    for i in s:
        dic[str(i)] = lis.count(i)

    return dic

def counter_count():
    # 方式4
    return collections.Counter(lis)


if __name__ == '__main__':
    print dict_count()                  # {'a': 3, 'c': 2, 'b': 1, '1': 2, '3': 2, '2': 3, '4': 1}
    print defaultdict_count()           # defaultdict(&lt;type 'int'&gt;, {'a': 3, 1: 2, 2: 3, 3: 2, 4: 1, 'c': 2, 'b': 1})
    print set_count()                   # {'a': 3, 'c': 2, 'b': 1, '1': 2, '3': 2, '2': 3, '4': 1}
    print counter_count()               # Counter({'a': 3, 2: 3, 1: 2, 3: 2, 'c': 2, 4: 1, 'b': 1})
</code></pre>

<p>上面使用了<code>collections</code>模块，其实用到的还是挺多的，这里就分析下这个模块吧</p>

<!--more-->

<h2>collections模块</h2>

<p>collections:High-performance container datatypes(高性能容器数据类型),<a href="https://docs.python.org/2/library/collections.html">文档地址</a></p>

<blockquote>
  <p>This module implements specialized container datatypes providing alternatives to Python’s general purpose built-in containers, dict, list, set, and tuple.</p>
</blockquote>

<h3>1. Counter对象</h3>

<pre><code>import collections
lis = ['a',1,2,3,4,1,2,3,'b','c','a','c','a',2]
cnt = collections.Counter()
print cnt       # Counter()

for word in lis:
    cnt[word] += 1

print cnt       # Counter({'a': 3, 2: 3, 1: 2, 3: 2, 'c': 2, 4: 1, 'b': 1})

print collections.Counter(lis)  # Counter({'a': 3, 2: 3, 1: 2, 3: 2, 'c': 2, 4: 1, 'b': 1})
print collections.Counter(lis).most_common(2)       #最常出现的2个 [('a', 3), (2, 3)]
</code></pre>

<p>源码是这样描述的：</p>

<pre><code>def __init__(self, iterable=None, **kwds):
        '''Create a new, empty Counter object.  And if given, count elements
        from an input iterable.  Or, initialize the count from another mapping
        of elements to their counts.

        &gt;&gt;&gt; c = Counter()                           # a new, empty counter
        &gt;&gt;&gt; c = Counter('gallahad')                 # a new counter from an iterable
        &gt;&gt;&gt; c = Counter({'a': 4, 'b': 2})           # a new counter from a mapping
        &gt;&gt;&gt; c = Counter(a=4, b=2)                   # a new counter from keyword args

        '''
</code></pre>

<p>不管哪种形式，我们得到的是<strong>字典接口</strong>的对象,可以像操作字典一样操作该对象，如果不存在该key,则返回0.</p>

<pre><code>c =  collections.Counter(dog=100, cat=200)
print c                 # Counter({'cat': 200, 'dog': 100})
print c['dog']          # 100
print c['gs']           # 0

c['gs'] = 1
print c                 # Counter({'cat': 200, 'dog': 100, 'gs': 1})
del c['dog']
print c                 # Counter({'cat': 200, 'gs': 1})
</code></pre>

<p>接下来介绍Counter对象的常用方法。</p>

<h4>1.1 elements()</h4>

<p>返回一个<code>iterator</code>对象, 根据计数生成多个值，如果计数为0则忽略</p>

<pre><code>c = collections.Counter(a=4, b=2, c=0, d=-2)
print c.elements()          # &lt;itertools.chain object at 0x1016fe890&gt;
print list(c.elements())    # ['a', 'a', 'a', 'a', 'b', 'b']
</code></pre>

<h4>1.2 most_common([n])</h4>

<p>返回n个最常出现的元素，如果n为None则返回所有</p>

<pre><code>print c.most_common()           # [('a', 4), ('b', 2), ('c', 0), ('d', -2)]

print c.most_common(2)          # [('a', 4), ('b', 2)]
</code></pre>

<h4>1.3 subtract([iterable-or-mapping])</h4>

<p>subtract是减掉的意思</p>

<pre><code>&gt;&gt;&gt; c = Counter(a=4, b=2, c=0, d=-2)
&gt;&gt;&gt; d = Counter(a=1, b=2, c=3, d=4)
&gt;&gt;&gt; c.subtract(d)
&gt;&gt;&gt; c
Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6})
</code></pre>

<p>我们也能直接使用<code>+</code>,<code>-</code>,<code>&amp;</code>,<code>|</code></p>

<pre><code>&gt;&gt;&gt; c = Counter(a=3, b=1)
&gt;&gt;&gt; d = Counter(a=1, b=2)
&gt;&gt;&gt; c + d                       # add two counters together:  c[x] + d[x]
Counter({'a': 4, 'b': 3})
&gt;&gt;&gt; c - d                       # subtract (keeping only positive counts)
Counter({'a': 2})
&gt;&gt;&gt; c &amp; d                       # intersection:  min(c[x], d[x])
Counter({'a': 1, 'b': 1})
&gt;&gt;&gt; c | d                       # union:  max(c[x], d[x])
Counter({'a': 3, 'b': 2})   
</code></pre>

<h3>2. defaultdict</h3>

<p>另一种最常用的就是<code>defaultdict</code>.</p>

<blockquote>
  <p>这里的defaultdict(function_factory)构建的是一个类似dictionary的对象，其中keys的值，自行确定赋值，但是values的类型，是function_factory的类实例，而且具有默认值。比如default(int)则创建一个类似dictionary对象，里面任何的values都是int的实例，而且就算是一个不存在的key, d[key] 也有一个默认值，这个默认值是int()的默认值0.</p>
</blockquote>

<pre><code>s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
d = collections.defaultdict(list)
print d             # defaultdict(&lt;type 'list'&gt;, {})
for k, v in s:
    d[k].append(v)

print d             # defaultdict(&lt;type 'list'&gt;, {'blue': [2, 4], 'red': [1], 'yellow': [1, 3]})

print list(d.items())   # [('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
</code></pre>

<p>如上设置function_factory 为list类型，由此指定了value应该以list身份出现。 我们可模拟如下：</p>

<pre><code>dic = {}
for k,v in s:
    dic[k] = list()        # 相当于  d = collections.defaultdict(list)
    for kk,vv in s:
        if kk == k:
            dic[k].append(vv)

print dic               # {'blue': [2, 4], 'red': [1], 'yellow': [1, 3]}
</code></pre>

<p>用了两次的循环才产生同上同样的效果。等等，还记得字典对象的<code>setdefault</code>方法么，它的效果也同上，那么就用它也写一个吧。</p>

<pre><code>dic = {}
for k, v in s:
    dic.setdefault(k, []).append(v)

print dic           # {'blue': [2, 4], 'red': [1], 'yellow': [1, 3]}
</code></pre>

<p>使用<strong>setdefault</strong>简洁了许多, <strong>collections.defaultdict(list)使用起来效果和运用dict.setdefault()比较相, 但是前者要更快一些，功能也更加强悍些。</strong></p>

<pre><code>&gt;&gt;&gt; s = 'mississippi'
&gt;&gt;&gt; d = defaultdict(int)
&gt;&gt;&gt; for k in s:
...     d[k] += 1
...
&gt;&gt;&gt; d.items()
[('i', 4), ('p', 2), ('s', 4), ('m', 1)]
</code></pre>
