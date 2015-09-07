---
layout: post
title: "python hashlib模块"
description: "python hashlib模块"
category: "python模块"
tags: [python模块]
---
{% include JB/setup %}

<p>该模块实现了一个通用的接口，许多不同的安全哈希和消息摘要的算法。包括的是FIPS安全散列算法SHA-1，SHA224，SHA256，SHA384，和SHA512（在FIPS180-2中定义）以及RSA的MD5算法（在因特网RFC1321中定义）。术语安全哈希和消息摘要是可互换的。旧的算法被称为消息摘要。现代的词是安全散列。</p>

<pre><code>import hashlib  
md5 = hashlib.md5()         #创建一个MD5加密对象,&lt;md5 HASH object @ 0x10e476630&gt; 
md5.update("beginman")      #更新要加密的数据  
print md5.digest()          #加密后的结果（二进制）  
print md5.hexdigest()       #加密后的结果，用十六进制字符串表示。  
print 'block_size:', md5.block_size  
print 'digest_size:', md5.digest_size  
</code></pre>

<p>简洁的方式：</p>

<pre><code>print hashlib.new("md5", "beginman").hexdigest()  
</code></pre>

<p>注意：</p>

<ol>
<li><code>m.update(a)</code>; <code>m.update(b)</code> is equivalent to <code>m.update(a+b)</code>.</li>
<li>加密不可逆</li>
<li>SHA1基于MD5，加密后的数据长度更长，比MD5更加安全，但SHA1的运算速度就比MD5要慢。</li>
<li>md5经常用来做用户密码的存储。而sha1则经常用作数字签名</li>
</ol>

<p><strong>应用场景：</strong></p>

<ul>
<li>密码处理</li>
<li>文件名唯一名称</li>
<li>KV数据库做key</li>
</ul>
