---
layout: post
title: "Python ConfigParser模块处理配置文件"
description: "Python ConfigParser模块处理配置文件"
category: "python模块"
tags: [python模块]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<blockquote>
  <p>配置文件的意义：不用改动代码（不用重新编译）就能改变应用程序的行为，使其更加灵活化，自由化。</p>
</blockquote>

<p><strong>常用的配置文件格式有:xml,ini文件</strong>，接下来主要学习ini文件，以及Python对配置文件的操作。</p>

<!--more-->

<h2>ini文件</h2>

<blockquote>
  <p>.ini 文件是Initialization File的缩写，即初始化文件，是windows的系统配置文件所采用的存储格式，统管windows的各项配置.</p>
</blockquote>

<p>当然ini文件也存在于Unix中，具有可移植性，但为了安全性，<strong>最好不要用这些后缀名，因为知道文件名时，在浏览器里输入该文件的地址时，可看到所有内容的。</strong></p>

<p>格式如下：</p>

<p><strong>INI文件由节(section)、键(key)、值(value)组成。</strong></p>

<p>节： [section]</p>

<p>参数（键=值）: key=value</p>

<p>注释：分号表示（;）</p>

<p>如下实例my.ini：</p>

<pre><code>; exp ini file
[port]
portname=COM4
port=4  
</code></pre>

<p>如果一个section中有多个相同的key则取最后一个。</p>

<h2>使用ConfigParser模块读写ini文件</h2>

<p>ConfigParser模块使用3个类(RawConfigParser、ConfigParser、SafeConfigParser)对ini文件进行读写。</p>

<p>本地建立一个my.ini文件用于测试</p>

<pre><code>[default]
name=beginman
age=24
address=Beijing
hobbit=Read Book,Coding,Play basketboll
</code></pre>

<h3>RawCnfigParser</h3>

<p>RawCnfigParser是最基础的INI文件读取类，<code>class ConfigParser.RawConfigParser([defaults[, dict_type[, allow_no_value]]])</code>, 默认defaults为None,dict_type为_default_dict，allow_no_value为False</p>

<pre><code>import ConfigParser
conf = ConfigParser.RawConfigParser()
conf.read('my.ini')
print conf.sections()                       # ['default']
print conf.items('default')                 # [('name', 'beginman'), ('age', '24'), ('address', 'Beijing'), ('hobbit', 'Read Book,Coding,Play basketboll')]
print conf.get('default', 'name')           # beginman
</code></pre>

<p>如果<code>defaults</code>给定了一个字典对象，则初始化内置字典附加my.ini处理。如果defaults中的key值与my.ini的key相同则忽略。</p>

<pre><code>conf = ConfigParser.RawConfigParser(defaults={'sex':'boy', 'hobbit':'sleeping'})
conf.read('my.ini')
print conf.items('default')                
# [('hobbit', 'Read Book,Coding,Play basketboll'), ('sex', 'boy'), ('name', 'beginman'), ('age', '24'), ('address', 'Beijing')]
</code></pre>

<p><strong>三者一致，推荐使用ConfigParser</strong>:</p>

<p><strong>基本的读取配置文件</strong>:</p>

<ul>
<li>read(filename) 直接读取ini文件内容</li>
<li>sections() 得到所有的section，并以列表的形式返回</li>
<li>options(section) 得到该section的所有option</li>
<li>items(section) 得到该section的所有键值对</li>
<li>get(section,option) 得到section中option的值，返回为string类型</li>
<li>getint(section,option) 得到section中option的值，返回为int类型</li>
<li>getboolean() "1", "yes", "true", and "on" 表示True, "0", "no", "false", and "off", 表示Flase</li>
<li>getfloat() 返回浮点型。</li>
</ul>

<p><strong>基本的写入配置文件</strong></p>

<ul>
<li>add_section(section) 添加一个新的section</li>
<li>set( section, option, value) 对section中的option进行设置，需要调用write将内容写入配置文件。</li>
<li>write(strout) 将对configparser类的修改写入</li>
</ul>

<p>如下实例：</p>

<pre><code>raw_conf = ConfigParser.RawConfigParser()
raw_conf.add_section('Add')
raw_conf.set('Add', 'jack', 'lll')
raw_conf.set('Add','foo', '%(bar)s is %(baz)s!')
with open('my.ini', 'a') as configfile:     # 追加方式
    raw_conf.write(configfile)
</code></pre>

<h3>ConfigParser</h3>

<p>ConfigParser由RawCnfigParser衍生，实现了一些魔法功能，并给<code>get()</code>和<code>items()</code>增加了可选参数.</p>

<h3>SafeConfigParser</h3>

<p>是ConfigParser的扩展，能插入一个更健全的变体(more-sane variant)</p>

<p>如下与<code>StringIO</code>的例子：</p>

<pre><code>from ConfigParser import SafeConfigParser
from StringIO import StringIO

f = StringIO()
scp = SafeConfigParser()

print '-'*20, ' following is write ini file part ', '-'*20
sections = ['s1','s2']
for s in sections:
    scp.add_section(s)
    scp.set(s,'option1','value1')
    scp.set(s,'option2','value2')

scp.write(f)
print f.getvalue()

print '-'*20, ' following is read ini file part ', '-'*20
f.seek(0)
scp2 = SafeConfigParser()
scp2.readfp(f)
sections = scp2.sections()
for s in sections:
    options = scp2.options(s)
    for option in options:
        value = scp2.get(s,option)
        print "section: %s, option: %s, value: %s" % (s,option,value)
</code></pre>

<p>输出如下：</p>

<pre><code>--------------------  following is write ini file part  --------------------
[s1]
option1 = value1
option2 = value2

[s2]
option1 = value1
option2 = value2


--------------------  following is read ini file part  --------------------
section: s1, option: option1, value: value1
section: s1, option: option2, value: value2
section: s2, option: option1, value: value1
section: s2, option: option2, value: value2
</code></pre>

<h3>异常</h3>

<ul>
<li>NoSectionError：当没有发现给定section时抛出。</li>
<li>DuplicateSectionError ：如果add_section()方法被调用时，提供的section参数的值已经存在时抛出。</li>
<li>NoOptionError ：指定option不存在时抛出。</li>
<li>InterpolationError：执行字符串填补时抛出的异常的基类。</li>
<li>InterpolationDepthError：当填补字符串因为迭代次数超过了MAX_INTERPOLATION_DEPTH值时抛出的异常，InterpolationError的子类。</li>
<li>InterpolationMissingOptionError 当option引用的值不存在时抛出，该异常为InterpolationError的子类，2.3版本新加。</li>
<li>InterpolationSyntaxError：当原文件格式没有遵守规定的语法时抛出的异常，继承至InterpolationError，2.3版本。</li>
<li>MissingSectionHeaderError：尝试解析没有section头的文件时抛出。</li>
<li>ParsingError：解析文件时发生错误。</li>
</ul>

<h3>技巧</h3>

<ol>
<li>最好在配置文件中设置DEFAULT节点， 因为如果ConfigParser在某一节点下没有找到key，那么就会去DEFAULT中找key。</li>
<li>支持字符串格式化行为</li>
</ol>

<p>如下是一个常用的SQLAlchemy 配置文件。</p>

<pre><code>[DEFAULT]
conn_str=%(dns) s//%(user) s:%(pass) s@%(host) s:%(port) s/%(db) s
[db1]
dns = mysql
user = root
pass = 12345
host = localhost
port = 3306
db = myblog
[db2]
dns = redis
user = root
pass = 1111
host = localhost
port = 6379
db = 3
</code></pre>

<p>我们在读取配置时可动态获取conn_str的值</p>

<pre><code>import ConfigParser
conf = ConfigParser.ConfigParser()
conf.read('my.ini')
print conf.get('db1', 'conn_str')           # mysql//root:12345@localhost:3306/myblog
print conf.get('db2', 'conn_str')           # redis//root:1111@localhost:6379/3
</code></pre>

<p>参考：《改善Python代码的91条建议》</p>
