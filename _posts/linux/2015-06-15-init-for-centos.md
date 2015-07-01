---
layout: post
title: "init for centos"
description: "init for centos"
category: "未分类"
tags: [未分类]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<pre><code>sudo yum install  bzip2-devel
yum install openssl openssl-devel
yum install zlib-devel
yum install python-devel

./configure
make &amp;&amp; make install
</code></pre>
