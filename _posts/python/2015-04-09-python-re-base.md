---
layout: post
title: "python正则表达式（1）：正则基础"
description: "python正则表达式（1）：正则基础"
category: "python"
tags: [python技巧]
---
{% include JB/setup %}

<p>本次对python正则进行系统学习，当然不限于一门编程语言，总体来说是对<code>正则</code>的学习与总结，这里分以下几个步骤：</p>

<h1>一.计划学习</h1>

<h2>1.正则原理</h2>

<p>由于有些正则基础，但总昏昏然，这次的学习要弄清楚正则的原理所在，如NFA引擎匹配原理，然后了解Python对处理正则的基本流程，这里参考学习链接如下：</p>

<!--more-->

<p><a href="http://blog.csdn.net/lxcnn/article/category/538256"><strong>1.正则基础</strong></a></p>

<h2>2. 正则表达式基础</h2>

<p>正则表达式元字符和语法，这里参考学习如下：</p>

<p><a href="http://blog.csdn.net/lxcnn/article/category/538256"><strong>1.正则基础</strong></a></p>

<p><a href="http://wiki.ubuntu.org.cn/Python%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%93%8D%E4%BD%9C%E6%8C%87%E5%8D%97"><strong>2.Python正则表达式操作指南</strong></a></p>

<p><a href="http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html"><strong>3.Python正则表达式指南</strong></a></p>

<h2>3.比较Python和Js正则处理机制</h2>

<p>一个后端，一个前段，都支持正则，虽然性质相同，但处理语法上不同，在学习中要给予总结。
参考学习：</p>

<p><a href="http://www.cnblogs.com/rubylouvre/archive/2010/03/09/1681222.html"><strong>1.javascript正则表达式</strong></a></p>

<p><a href="http://www.iteye.com/topic/481228"><strong>2.精通 JS正则表达式</strong></a></p>

<h2>4.实战演练</h2>

<p><strong>1.处理常用的验证工作</strong>，如<a href="经典JavaScript正则表达式实战">这里</a></p>

<p><strong>2.Python爬虫处理</strong></p>

<h1>二.学习总结</h1>

<h2>2.1 正则基础</h2>

<h3>1.[]字符组</h3>

<h4>(1)基础概念</h4>

<p>[]字符组只能匹配一个字符[]字符组只能匹配一个字符，里面包含一系列<code>或</code>的字符，如<code>[abc]</code>,表示匹配字符a或b或c。因此在[]中使用“|”来表示“或”的关系是错误的,如[a|b|c]就不对了。</p>

<pre><code>import re
m = re.match(r'[abc]', 'abcdefg')
if m: print m.group()    # 输出 a, 说明只匹配一个

# JS
console.log('abcdefg'.match(/[abc]/))
["a", index: 0, input: "abcdefg", contain: function]
</code></pre>

<h4>(2)连字符<code>-</code></h4>

<p>连字符<code>-</code> 用在<code>[]</code>中表示一个范围，一个区间，这就表明大小顺序的位置了，如<code>[a-z]</code>表示小写字母，但不能写成<code>[z-a]</code>.</p>

<h4>(3)特殊性</h4>

<p>在<code>[]</code>的特殊字符不需要转义，除了<code>\</code>,<code>[]</code>,<code>-</code>,<code>^</code>,<code>$</code>之外。如：</p>

<pre><code>console.log('abcdefg'.match(/[abc]/))
["a", index: 0, input: "abcdefg", contain: function]
</code></pre>

<p>对于<code>[^]则表示不包含</code></p>

<pre><code># [\u4e00-\u9fa5]表示任意一个汉字
console.log('你好'.match(/[\u4e00-\u9fa5]/))
["你", index: 0, input: "你好", contain: function]
</code></pre>

<h3>2.字符范围缩写</h3>

<p>具体知识点，参考这里<a href="http://blog.csdn.net/lxcnn/article/details/4268033">http://blog.csdn.net/lxcnn/article/details/4268033</a>.注意点，没必要死记硬背，如d:digital [0-9];  w:word [a-zA-Z0-9_]; s:space [ \r\n\f\t\v ], 只要是大写如：D,S,W就表示其反义。</p>

<h3>3.小数点<code>.</code></h3>

<p>匹配除<code>\n</code>外任意<strong>一个字符</strong>，如果匹配全部则使用<code>[\s\S]</code></p>

<blockquote>
  <p>对于使用传统NFA引擎的大多数语言，如Java，.NET来说，“.”的匹配范围是这样的。 但是对于JavaScript来说有些特殊，由于各浏览器的解析引擎不同，“.”的匹配范围也有所不同，对于Trident内核的浏览器，如IE来说，“.”同样是匹配除了换行符“\n”以外的任意一个字符，但是对于其它内核的浏览器，如Firefox、Opera、Chrome来说，“.”是匹配除了回车符“\r”和换行符“\n”以外的任意一个字符。</p>
</blockquote>

<pre><code>//js
s = '&lt;td&gt;This is a test line.\
              Another line. &lt;/td&gt;'

s.match(/&lt;td&gt;[\s\S]*&lt;\/td&gt;/)
["&lt;td&gt;This is a test line.              Another line. &lt;/td&gt;"]
</code></pre>

<h3>4.量词</h3>

<p>主要表示匹配的次数，具体内容参考上个链接。注意，不要出现<code>{1}</code>这样的量词，这和<code>\w</code>一样，但是上个能降低可读性和匹配效率。</p>

<h3>5.分支结构</h3>

<p>分支结构就是<code>|</code>在多个字表达式中取“或”，<code>|</code>以括号<code>()</code>为限定，如在()中则匹配范围限制在（）内，否则为<code>|</code>左右两侧整体，如：</p>

<pre><code>console.log('cccb'.match(/^aa|b$/))
["b", index: 3, input: "cccb", contain: function]
console.log('cccb'.match(/^(aa|b)$/))
null
</code></pre>

<h3>5.捕获组</h3>

<p>捕获组就是正则匹配的内容暂时保存到内存中，这段内存是有编号的（1,2..）或有名称的(手动为捕获组命名)</p>

<blockquote>
  <p>1.(Expression) : 普通捕获组将字表达式Expression捕获的内容添加到该编号的组里面。0编号表示整个表达式的组。</p>
  
  <p>2.(?<name>Expression): 命名捕获组，将子表达式Expression捕获的内容存放在名为name的命名组里。</p>
  
  <p>对于Python,其语法规则如下：<strong>Python 新增了一个扩展语法到 Perl 扩展语法中。如果在问号后的第一个字符是 "P"，你就可以知道它是针对 Python 的扩展。目前有两个这样的扩展: (?P<name>...) 定义一个命名组，(?P=name) 则是对命名组的逆向引用。</strong>,所以它这样写<code>(?P&lt;name&gt;Expression)</code>.</p>
</blockquote>

<p>在Python/Django 中，常见的路由映射，就大量运用到了捕获组，如下：</p>

<pre><code>(r'^see/(?P&lt;uid&gt;\d+)/(?P&lt;unit_class_id&gt;\d+)/(?P&lt;book_id&gt;\d+)/$', 'view1'),    
</code></pre>

<p>这段路由中，前段实则是正则表达式，它匹配以'see'开头后面是/数字的字符串，并将后面匹配的三个数字保存到命名组为uid,unit_class_id,book_id里面，那么在视图函数中就要反向引用这几个命名组的名称了，如下：</p>

<pre><code>def view1(request, uid, unit_class_id, book_id):
    uid = uid  # 路由正则里命名组uid的值
    unit_class_id = unit_class_id  # ....
    .....
</code></pre>

<p>通过上述内容，可以清晰了解Django的路由规则。</p>

<p><strong>在Python中，使用捕获组如同使用字典般方便，如下：</strong></p>

<pre><code>import re
s = 'BeginMan:120'
m = re.compile(r'(?P&lt;name&gt;\w+):(?P&lt;age&gt;\d+)')
ms =  m.match(s)
if ms:
    print ms.group()
    print ms.groups()
    print ms.groupdict()
    print ms.group('name')
    print ms.group('age')

# output:
BeginMan:120
('BeginMan', '120')
{'age': '120', 'name': 'BeginMan'}
BeginMan
120
</code></pre>

<h3>6.非捕获组</h3>

<blockquote>
  <p>一些表达式中，不得不使用(  )，但又不需要保存(  )中子表达式匹配的内容，这时可以用非捕获组来抵消使用(  )带来的副作用。创建一个非捕获性分组，只要在分组的左括号的后面紧跟一个问号与冒号就行了。</p>
</blockquote>

<pre><code>var color = '#999000'
/#(\d+)/.test(color)  # 捕获组
console.log(RegExp.$1)  # 输出 999000
console.log(RegExp.$0)  # 无输出

/#(?:\d+)/.test(color)  # 非捕获组
console.log(RegExp.$1)  # 无输出
console.log(RegExp.$0)  # 无输出
</code></pre>

<h3>7.环视（零宽断言）</h3>

<p>在<a href="http://blog.csdn.net/lxcnn/article/details/4268033">正则表达式学习参考</a>这篇文章中，对字符串进行说明，如:</p>

<p><img src="http://hi.csdn.net/attachment/201002/9/35916_12656741378Q86.jpg" alt="" /></p>

<p>上述字符串a5由字符和位置构成，对于理解正则表达式原理很重要。</p>

<h4>占有字符和零宽度：</h4>

<blockquote>
  <p>正则表达式匹配过程中，如果子表达式匹配到的是字符内容，而非位置，并被保存到最终的匹配结果中，那么就认为这个子表达式是占有字符的；如果子表达式匹配的仅仅是位置，或者匹配的内容并不保存到最终的匹配结果中，那么就认为这个子表达式是零宽度的。
  占有字符还是零宽度，是针对匹配的内容是否保存到最终的匹配结果中而言的。
  占有字符是互斥的，零宽度是非互斥的。也就是一个字符，同一时间只能由一个子表达式匹配，而一个位置，却可以同时由多个零宽度的子表达式匹配。</p>
</blockquote>

<p><code>环视</code>，又叫<code>零宽断言</code>，或<code>前瞻后瞻</code>，在js中不支持后瞻，也就是逆向环视。在Python中都支持。
这里推荐阅读<a href="http://blog.csdn.net/magictong/article/details/5332423">Py正则表达式中的【零宽断言】</a>.</p>

<pre><code>#!/usr/bin/env python
# coding=utf-8
import re
s = 'aaa111aaa , bbb222&amp;, 333ccc'
re_1 = r'\d+(?=[a-z]+)'   # 正向界定，(?=expression),表示要匹配右边的表达式
re_2 = r'\d+(?![a-z]+)'   # 正向否定界定，(?!expression),表示右边不匹配表达式
re_3 = r'(?&lt;=[a-z])\d+'    # 逆向界定，(?&lt;=expression),表示左边要匹配表达式，且表达式express只允许定长文本，（不能 r'(?&lt;=[a-z]+)\d+'）
re_4 = r'(?&lt;![a-z])\d+'    # 逆向否定界定，(?&lt;!expression),表示左边不匹配表达式

print re.findall(re_1, s)
print re.findall(re_2, s)
print re.findall(re_3, s)
print re.findall(re_4, s)

#output:
['111', '333']
['11', '222', '33']
['111', '222']
['11', '22', '333']
</code></pre>

<h3>8.贪婪与非贪婪匹配</h3>

<p>非贪婪就是在贪婪字符后面加<code>?</code>，表示尽量少的去匹配。如：</p>

<pre><code>s= 'abc123';
console.log(st.match(/abc\d+/))
["abc123", index: 0, input: "abc123", contain: function]
console.log(st.match(/abc\d+?/))
["abc1", index: 0, input: "abc123", contain: function]
</code></pre>

<h3>9.反向引用</h3>

<p>对<strong>捕获组</strong>内容引用，可在正则表达式外和内引用，在之内的引用就叫做<strong>反向引用</strong>。通常以<code>\</code>开头，后面跟编号或名称，如：</p>

<p><code>\1</code>: 对编号为1的捕获组的反向引用。</p>

<p><code>\k&lt;name&gt;</code>： 对名称为name的捕获组的反向引用</p>

<pre><code>// js
var color = "#990000";
/#(\d+)/.test(color);
alert(RegExp.$1);//990000

'abaa445566'.match(/(a|b)\1(\d)\2/)
["aa44", "a", "4"]

# py(待定...)
</code></pre>

<h2>2.2 捕获组的总结</h2>

<blockquote>
  <p>在.NET中使用(?’name’Expression)与使用(?<name>Expression)是等价的。在PHP和Python中命名捕获组语法为：(?P<name>Expression)。另外需要说明的一点是，除(Expression)和(?<name>Expression)语法外，其它的(?...)语法都不是捕获组。</p>
</blockquote>

<p>参考链接<a href="http://blog.csdn.net/lxcnn/article/details/4146148">http://blog.csdn.net/lxcnn/article/details/4146148</a></p>

<h3>1.编号规则</h3>

<p>如果没有指定名称，则默认的编号从左到右以1开始。如下图<code>(\d{4})-(\d{2}-(\d\d))</code>匹配格式为yyyy-MM-dd的日期：</p>

<p><img src="http://hi.csdn.net/attachment/201002/8/35916_12656733930M0l.jpg" alt="" /></p>

<table>
<thead>
<tr>
  <th>编号</th>
  <th>名称</th>
  <th>捕获组</th>
  <th>匹配内容</th>
</tr>
</thead>
<tbody>
<tr>
  <td>0</td>
  <td></td>
  <td><code>(\d{4})-(\d{2}-(\d\d))</code></td>
  <td>2014-07-01</td>
</tr>
<tr>
  <td>1</td>
  <td></td>
  <td><code>(\d{4})</code></td>
  <td>2014</td>
</tr>
<tr>
  <td>2</td>
  <td></td>
  <td><code>(\d{2}-(\d\d))</code></td>
  <td>07-01</td>
</tr>
<tr>
  <td>3</td>
  <td></td>
  <td><code>(\d\d)</code></td>
  <td>01</td>
</tr>
</tbody>
</table>

<pre><code>//js
'2014-07-01'.match(/(\d{4})-(\d{2}-(\d\d))/)
["2014-07-01", "2014", "07-01", "01"]
</code></pre>

<h3>2.命名编号规则</h3>

<p>对命名捕获组其编号规则也是从左到右，如<code>(?&lt;year&gt;\d{4})-(?&lt;date&gt;\d{2}-(?&lt;day&gt;\d\d))</code></p>

<p><img src="http://hi.csdn.net/attachment/201002/8/35916_1265673394X74F.jpg" alt="" /></p>

<table>
<thead>
<tr>
  <th>编号</th>
  <th>名称</th>
  <th>捕获组</th>
  <th>匹配内容</th>
</tr>
</thead>
<tbody>
<tr>
  <td>0</td>
  <td></td>
  <td><code>(?&lt;year&gt;\d{4})-(?&lt;date&gt;\d{2}-(?&lt;day&gt;\d\d))</code></td>
  <td>2014-07-01</td>
</tr>
<tr>
  <td>1</td>
  <td>year</td>
  <td><code>(?&lt;year&gt;\d{4})</code></td>
  <td>2014</td>
</tr>
<tr>
  <td>2</td>
  <td>date</td>
  <td><code>(?&lt;date&gt;\d{2}-(?&lt;day&gt;\d\d))</code></td>
  <td>07-01</td>
</tr>
<tr>
  <td>3</td>
  <td>day</td>
  <td><code>(?&lt;day&gt;\d\d)</code></td>
  <td>01</td>
</tr>
</tbody>
</table>

<pre><code># py
&gt;&gt;&gt; r = r'(?P&lt;year&gt;\d{4})-(?P&lt;date&gt;\d{2}-(?P&lt;day&gt;\d\d))'
&gt;&gt;&gt; re.findall(r, '2014-07-01')
[('2014', '07-01', '01')]
</code></pre>

<h3>3.混合编号规则</h3>

<p>既有命名捕获组又有编号捕获组，这个时候优先以编号捕获组查找，忽略命名捕获组，然后再按命名捕获组查找。如下：<code>(\d{4})-(?&lt;date&gt;\d{2}-(\d\d))</code></p>

<p><img src="http://hi.csdn.net/attachment/201002/8/35916_126567339559Fx.jpg" alt="" /></p>

<table>
<thead>
<tr>
  <th>编号</th>
  <th>名称</th>
  <th>捕获组</th>
  <th>匹配内容</th>
</tr>
</thead>
<tbody>
<tr>
  <td>0</td>
  <td></td>
  <td><code>(\d{4})-(?&lt;date&gt;\d{2}-(\d\d))</code></td>
  <td>2014-07-01</td>
</tr>
<tr>
  <td>1</td>
  <td></td>
  <td><code>(\d{4})</code></td>
  <td>2014</td>
</tr>
<tr>
  <td>3</td>
  <td></td>
  <td><code>(\d\d)</code></td>
  <td>01</td>
</tr>
<tr>
  <td>2</td>
  <td>date</td>
  <td><code>(?&lt;date&gt;\d{2}-(?&lt;day&gt;\d\d))</code></td>
  <td>07-01</td>
</tr>
</tbody>
</table>

<pre><code>#py
&gt;&gt;&gt; r = r'(\d{4})-(?P&lt;date&gt;\d{2}-(\d\d))'
&gt;&gt;&gt; re.findall(r, '2014-07-01')
[('2014', '07-01', '01')]
</code></pre>

<h1>三.手册与工具</h1>

<p><a href="http://www.php100.com/manual/unze.html"><strong>1.正则手册</strong></a></p>

<p><a href="http://tool.chinaz.com/regex/"><strong>2.正则表达式在线测试</strong></a></p>

<p><a href="http://t.mb5u.com/jquery/regexp.html"><strong>3.正则表达式速查表</strong></a></p>

<p><strong>速查图</strong></p>

<p><img src="http://images.cnblogs.com/cnblogs_com/huxi/Windows-Live-Writer/Python_10A67/pyre_ebb9ce1c-e5e8-4219-a8ae-7ee620d5f9f1.png" alt="" /></p>
