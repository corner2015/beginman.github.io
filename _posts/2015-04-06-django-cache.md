---
layout: post
title: "django缓存系统之django缓存"
description: "django缓存系统之django缓存"
category: "django"
tags: [django]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<h1>一.django中几种设置缓存的方式</h1>

<p>首先要通过配置<code>CACHES</code>,<a href="https://docs.djangoproject.com/en/1.5/ref/settings/#std:setting-CACHES">CACHES</a>来告诉系统缓存存放在哪里,是数据库中,文件系统还是直接放在内存中.接下来介绍几种缓存方式.</p>

<h2>1.Memcached</h2>

<blockquote>
  <p>Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。</p>
</blockquote>

<p><img src="http://memcached.org/images/memcached_banner75.jpg" alt="" /></p>

<p>文档中说Memecached是迄今为止用于Django中最快最有效的缓存类型,这里尚且不确定,后期时间我会比较<code>Redis</code>和<code>Memcached</code>在Django中的性能.</p>

<!--more-->

<p>对于python的支持,这里有两种客户端 python-memcached 和 pylibmc.这里选择第一种试试.</p>

<pre><code>#安装
sudo apt-get install memcached

#启动
-d 选项是启动一个守护进程
-m 是分配给Memcache使用的内存数量，单位是MB，默认64MB
-M return error on memory exhausted (rather than removing items)
-u 是运行Memcache的用户，如果当前为root 的话，需要使用此参数指定用户
-l 是监听的服务器IP地址，默认为所有网卡
-p 是设置Memcache的TCP监听的端口，最好是1024以上的端口
-c 选项是最大运行的并发连接数，默认是1024
-P 是设置保存Memcache的pid文件
-f chunk size growth factor (default: 1.25)
-I Override the size of each slab page. Adjusts max item size(1.4.2版本新增)

#启动例子
/usr/bin/memcached -d -m 100 -c 1000 -u root -p 11211
</code></pre>

<p>然后安装配置python-memcached.</p>

<pre><code>#安装
pip install python-memcached

#操作
&gt;&gt;&gt; import memcache
&gt;&gt;&gt; mc = memcache.Client(['127.0.0.1:11211'], debug=True)
&gt;&gt;&gt; mc.set('name', 'BeginMan', 60)
True
&gt;&gt;&gt; print mc.get('name')
BeginMan
&gt;&gt;&gt; mc.delete('name')
1
</code></pre>

<p>django中配置memcached.</p>

<p>(1).根据你所选择上述的两者来设置 <code>BACKEND</code>:django.core.cache.backends.memcached.MemcachedCache 或者 django.core.cache.backends.memcached.PyLibMCCache</p>

<p>(2).设置<code>LOCATION</code> IP:port值.这里要对应系统中启动的memcached的ip和端口,见上述.或者是<code>unix:path</code>值,这里要对应Memcached在linux中socket文件路径.</p>

<p>Memcached的一个极好的特性是它在多个服务器间分享缓存的能力。 这意味着您可以在多台机器上运行Memcached的守护进程，该程序会把这些机器当成一个单一缓存，而无需重复每台机器上的缓存值。要充分利用此功能，请在LOCATION里引入所有服务器的地址，以List形式.</p>

<pre><code>#django Memcached配置的几种方式
#方式1:
CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
    'LOCATION': '127.0.0.1:11211',            
        }
    }

#方式2:
CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
    'LOCATION': 'unix:/usr/bin/memcached.sock',            
        }
    }

#方式3:
CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
    'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
            ]
        }
    }
</code></pre>

<p><strong>注意:Memcached不能持久化,当服务器重启后,内存的东西都没了</strong></p>

<h2>2.数据库缓存</h2>

<p>通过数据库表来作为缓存后端,文档中介绍的是通过常用的关系型数据库如Mysql实现的,我们也可以完全通过redis来实现高性能的数据库缓存系统.先来看看文档中说明吧.</p>

<p>首先创建缓存表,当然这个数据库一定是你django项目对应的数据库了.</p>

<pre><code>python manage.py createcachetable [cache_table_name]
</code></pre>

<p>然后配置settings.py中CACHES:</p>

<pre><code>CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
    'LOCATION': 'my_cache_table',        
    }
}
</code></pre>

<p>对于多数据库缓存参考<a href="https://docs.djangoproject.com/en/1.5/topics/cache/#database-caching-and-multiple-databases">文档:Database caching and multiple databases</a></p>

<h2>3.文件系统缓存</h2>

<p>这个配置起来很简单</p>

<pre><code>CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
    'LOCATION': '/var/tmp/django_cache',        
    }
}
</code></pre>

<p>注意:要确保你配置的文件可读写,每个缓存值将被存储为单独的文件，其内容是Python的pickle模块以序列化(“pickled”)形式保存的缓存数据。 每个文件的名称是缓存键，以规避开安全文件系统的使用。</p>

<h2>4.本地内存缓存</h2>

<p>如果你想利用内存缓存的速度优势，但又不能使用Memcached，可以考虑使用本地存储器缓存后端。 此缓存的多进程和线程安全。</p>

<pre><code>CACHES = {
    'default': {
    'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    'LOCATION': 'unique-snowflake'        
    }
}
</code></pre>

<p>不过并不推荐这样处理,我也没处理过..</p>

<h1>二.Cache参数</h1>

<p>1.<code>TIMEOUT</code>:超时时间,秒为单位,默认300s</p>

<p>2.<code>OPTIONS</code>:CACHE后端选项,<code>MAX_ENTRIES</code>表示最大量,当超过这个量后会被删除,默认300.<code>CULL_FREQUENCY</code>表示删除的比例,默认是3,表示删除的比例是1/3, 0 意味着当达到 max_entries 时,缓存将被清空.</p>

<p>这里列举主要参数,更多参数参考文档.</p>

<pre><code>CACHES = {
    'default': {
                'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
                'LOCATION': '/var/tmp/django_cache',
                'TIMEOUT': 60,
                'OPTIONS': {
                        'MAX_ENTRIES': 1000
        }

    }

}
</code></pre>

<h1>三.站点级缓存</h1>

<p>一旦加入缓存,我们缓存的是整站的,要在中间件中这样处理:</p>

<pre><code>MIDDLEWARE_CLASSES = (
    'django.middleware.cache.UpdateCacheMiddleware',     #第一
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',  #最后
)
</code></pre>

<p>然后在settings.py中加入如下:</p>

<p>CACHE_MIDDLEWARE_ALIAS: 用于存储缓存的别名,默认是default.</p>

<p>CACHE_MIDDLEWARE_SECONDS: 每个页面应该被缓存的秒数。</p>

<p>CACHE_MIDDLEWARE_KEY_PREFIX ：如果缓存被多个使用相同Django安装的网站所共享，那么把这个值设成当前网站名，或其他能代表这个Django实例的唯一字符串，以避免key发生冲突。 如果你不在意的话可以设成空字符串。</p>

<p>缓存中间件缓存每个没有GET或者POST参数的页面。 或者，如果CACHE_MIDDLEWARE_ANONYMOUS_ONLY设置为True，只有匿名请求（即不是由登录的用户）将被缓存。 如果想取消用户相关页面（user-specific pages）的缓存，例如Djangos 的管理界面，这是一种既简单又有效的方法。 CACHE_MIDDLEWARE_ANONYMOUS_ONLY，你应该确保你已经启动AuthenticationMiddleware。</p>

<p>此外，缓存中间件为每个HttpResponse自动设置了几个头部信息：</p>

<p>当一个新(没缓存的)版本的页面被请求时设置Last-Modified头部为当前日期/时间。</p>

<p>设置Expires头部为当前日期/时间加上定义的CACHE_MIDDLEWARE_SECONDS。</p>

<p>设置Cache-Control头部来给页面一个最长的有效期，值来自于CACHE_MIDDLEWARE_SECONDS设置。</p>

<p>更多内容参考<a href="https://docs.djangoproject.com/en/1.5/topics/http/middleware/">中间件</a></p>

<h1>四.视图级别缓存</h1>

<p>如下将my_view函数缓存15分钟.</p>

<pre><code>from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def my_view(request):
    ...
</code></pre>

<p>和站点缓存一样，视图缓存与 URL 无关。 如果多个 URL 指向同一视图，每个视图将会分别缓存。 如:</p>

<pre><code>urlpatterns = ('',
    (r'^foo/(\d{1,2})/$', my_view),
)
</code></pre>

<p>且<strong>该方法对视图和缓存进行耦合了,并不太理想</strong></p>

<h1>五.URLconf 中指定视图缓存</h1>

<p>如下处理能解决视图缓存硬编码问题.</p>

<pre><code>from django.views.decorators.cache import cache_page

urlpatterns = ('',
    (r'^foo/(\d{1,2})/$', cache_page(60 * 15)(my_view)),
)
</code></pre>

<h1>六.模板碎片缓存</h1>

<p>可以使用<code>cache标签</code>来缓存模板片段。 在模板的顶端附近加入<code>{% load cache %}</code>以通知模板存取缓存标签。
模板标签{% cache %}在给定的时间内缓存了块的内容。 它至少需要两个参数: 缓存超时时间（以秒计）和指定缓存片段的名称。 示例：</p>

<pre><code>{% load cache %}
{% cache 500 sidebar %}
    .. sidebar ..
{% endcache %}
</code></pre>

<p>有时你可能想缓存基于片段的动态内容的多份拷贝。 比如，你想为上一个例子的每个用户分别缓存侧边栏。 这样只需要给{% cache %}传递额外的参数以标识缓存片段。</p>

<pre><code>{% load cache %}
{% cache 500 sidebar request.user.username %}
    .. sidebar for logged in user ..
{% endcache %}
</code></pre>

<p>传递不止一个参数也是可行的。 简单地把参数传给{% cache %}。</p>

<p>缓存超时时间可以作为模板变量，只要它可以解析为整数值。 例如，如果模板变量my_timeout值为600，那么以下两个例子是等价的。</p>

<pre><code>{% cache 600 sidebar %} ... {% endcache %}
{% cache my_timeout sidebar %} ... {% endcache %}
</code></pre>

<p>这个特性在避免模板重复方面非常有用。 可以把超时时间保存在变量里，然后在别的地方复用。</p>

<p>如果配置了<code>i18n</code>( internationalization的首末字符i和n，18为中间的字符数,是“国际化”的简称),可以这样处理</p>

<pre><code>{% load i18n %}
{% load cache %}

{% get_current_language as LANGUAGE_CODE %}

{% cache 600 welcome LANGUAGE_CODE %}
    {% trans "Welcome to example.com" %}
{% endcache %}
</code></pre>

<p>还有些低级别的缓存api,可参考文档.</p>

<p># 七.参考
 <a href="https://docs.djangoproject.com/en/1.5/topics/cache/#database-caching-and-multiple-databases">1.Django’s cache framework</a></p>

<p><a href="http://djangobook.py3k.cn/2.0/chapter15/">2.http://djangobook.py3k.cn/2.0/chapter15/</a>(注:这个太老了.)</p>
