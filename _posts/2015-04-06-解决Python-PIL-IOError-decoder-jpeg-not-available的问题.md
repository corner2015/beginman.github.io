---
layout: post
title: "解决Python PIL IOError: decoder jpeg not available的问题"
description: "解决Python PIL IOError: decoder jpeg not available的问题"
category: "Python"
tags: [Python]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p>mac:</p>

<pre><code>brew update
brew install libjpeg libpng
</code></pre>

<p>Ubuntu:</p>

<pre><code>sudo apt-get install libjpeg-dev
</code></pre>

<p>then:</p>

<pre><code>pip install -I PIL
</code></pre>

<p>Reference:<a href="http://geekforbrains.com/how-to-fix-the-decoder-jpeg-not-available-error-when-using-pil-in-python">http://geekforbrains.com/how-to-fix-the-decoder-jpeg-not-available-error-when-using-pil-in-python</a></p>
