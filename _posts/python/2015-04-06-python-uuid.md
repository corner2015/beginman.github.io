---
layout: post
title: "python uuid生成伪随机数"
description: "python uuid生成伪随机数"
category: "Python"
tags: [Python]
---
{% include JB/setup %}

<p>uuid也叫GUID,(UUID —— Universally Unique IDentifier；GUID —— Globally Unique IDentifier) 全局唯一标识符，128位个长二进制整数， 具体参考：<a href="http://zh.wikipedia.org/wiki/%E5%85%A8%E5%B1%80%E5%94%AF%E4%B8%80%E6%A0%87%E8%AF%86%E7%AC%A6">GUID</a></p>

<p><a href="https://docs.python.org/2/library/uuid.html">document</a></p>

<pre><code>&gt;&gt;&gt; uuid.uuid1()
UUID('7df790cc-6a4e-11e4-9707-9cf387a6e308')
&gt;&gt;&gt; uuid.uuid4()
UUID('87a31a3c-4819-462d-9a35-91d944b0bdad')
&gt;&gt;&gt; uuid.uuid3(uuid.NAMESPACE_DNS,'222')
UUID('c6b6add2-965d-3895-8397-14908b2d7a5e')

没有uuid2()
</code></pre>
