---
layout: post
title: "Redis+Django(Session,Cookie)的用户系统"
description: "Redis+Django(Session,Cookie)的用户系统"
category: "redis"
tags: [redis]
---
{% include JB/setup %}
<h1>一.Django authentication</h1>

<p><a href="https://docs.djangoproject.com/en/1.5/topics/auth/">django authentication</a>提供了一个便利的user api接口,无论在py中 <code>request.user</code>,参见<a href="https://docs.djangoproject.com/en/1.5/ref/request-response/">Request and response objects</a>.还是模板中的<code>{{user}}</code>都能随时随地使用,如果从web开发角度来看,其实无非就是cookie与session的运用.</p>

<p>在项目首页,在登陆和注销状态下分别输出所有session,如:</p>

<pre><code>print request.session.items()
# 登陆状态下输出
[('domain', 'http://beginman.sinaapp.com'), ('_auth_user_backend', 'django.contrib.auth.backends.ModelBackend'), ('_auth_user_id', 1L)]
# 注销状态下输出
[('domain', 'http://beginman.sinaapp.com')]
</code></pre>

<!--more-->

<p>从输出结果中可知晓,如果项目中settings.py配置如下:</p>

<pre><code>#中间件
MIDDLEWARE_CLASSES = (
        'django.middleware.common.CommonMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware', #看这里
        # 'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware', #看这里
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.gzip.GZipMiddleware',    # 处理gzip压缩,减轻服务器压力
        'pagination.middleware.PaginationMiddleware',  # django 第三方分页
        'common.mymiddleware.Mymiddleware',
        # Uncomment the next line for simple clickjacking protection:
        # 'django.middleware.clickjacking.XFrameOptionsMiddleware',

)

#TEMPLATE_CONTEXT_PROCESSORS
# 注意django1.5的这个玩意儿与低版本的不同
# 参考：https://docs.djangoproject.com/en/1.3/ref/settings/#std:setting-TEMPLATE_CONTEXT_PROCESSORS
TEMPLATE_CONTEXT_PROCESSORS = (
    "django.contrib.auth.context_processors.auth",
    "django.core.context_processors.debug",
    "django.core.context_processors.i18n",
    "django.core.context_processors.media",
    "django.core.context_processors.static",
    "django.core.context_processors.request",
    "django.contrib.messages.context_processors.messages"
)

    INSTALLED_APPS = (
    'django.contrib.auth',   #this
    'django.contrib.contenttypes', #this
    'django.contrib.sessions', #this
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'mysite',
)
</code></pre>

<p>那么就会有这样的输出,当然这只是针对单用户(这里拿我的博客开刀).在登陆后同时在首页输出cookies:</p>

<pre><code>print request.COOKIES
{'csrftoken': '9fBE9Kh0uuzXEMzWdc4z4aIOoZg1EaoI', 'sessionid': 'lf4dd7xjlyzrh4yvzbtltlbujy3ipp1f', 'Hm_lvt_c65358e73ce306691a49ae5119f58783': '1405408338'}
</code></pre>

<p>登陆成功后Django会自动在客户端生成一个sessionid,且这个sessionid在未注销前一直不变,当注销后就改变了该sessionid了. Cookie中保存一个Session的索引编号(sessionid)，其重要信息都保存在服务器端,Session控制.即可.</p>

<p>通过Cookie保存的sessionid与服务器端比较,当等于时则表示用户已登陆,若不等于或两者有一方不存在或都不存在则用户处于注销状态.</p>

<h1>二.Session 与Cookie</h1>

<p><img src="http://hi.csdn.net/attachment/201111/17/0_132156434454Ee.gif" alt="" /></p>

<p>图片来源:http://blog.csdn.net/tzjly/article/details/6986268</p>

<p>cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案,由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的.</p>

<p>关于session和cookie,这里有篇文章说的很清楚,<a href="http://www.cnblogs.com/shiyangxt/articles/1305506.html">传送门</a>.</p>

<h1>三.Django对于Cookie的处理方式</h1>

<p>每个<code>HttpRequest</code>对象都对应一个<code>COOKIES</code>对象,该对象是字典形式.</p>

<pre><code>request.COOKIES['sessionid']  # 获取
request.COOKIES.get('sessionid', None) # 获取
</code></pre>

<p>对COOKIES的设置通过<code>HttpResponse</code>对象的<code>set_cookie</code>来完成,<a href="https://docs.djangoproject.com/en/1.5/ref/request-response/#django.http.HttpResponse.set_cookie">文档传送门</a></p>

<pre><code>HttpResponse.set_cookie(key, value='', max_age=None, expires=None, path='/', domain=None, secure=None, httponly=False)
</code></pre>

<p>参数如下:</p>

<p>(1).`max_age<code>默认:None ,cookie需要延续的时间（以秒为单位） 如果参数是\</code> None`` ，这个cookie会延续到浏览器关闭为止。</p>

<p>(2).`expires<code>默认None ,cookie失效的实际日期/时间。 它的格式必须是：\</code> "Wdy, DD-Mth-YY HH:MM:SS GMT"<code>。如果给出了这个参数，它会覆盖\</code> max_age`` 参数。</p>

<p>(3).<code>path</code> 默认是<code>"/"</code>  ,cookie生效的路径前缀。 浏览器只会把cookie回传给带有该路径的页 面，这样你可以避免将cookie传给站点中的其他的应用,当你不是控制你的站点的顶层时，这样做是特别有用的。</p>

<p>(4).<code>domain</code> 默认None,这个cookie有效的站点。 你可以使用这个参数设置一个跨站点（cross-domain）的cookie。 比如，\ <code>domain=".example.com"</code> 可以设置一个在\ <code>www.example.com</code> 、\ <code>www2.example.com</code> 以及\ <code>an.other.sub.domain.example.com</code> 站点下都可读到的cookie。
   如果这个参数被设成\ <code>None</code> ，cookie将只能在设置它的站点下可以读到。</p>

<p>(5).<code>False</code> 默认False ,如果设置为 True ，浏览器将通过HTTPS来回传cookie。</p>

<h1>四.Django对Session的处理</h1>

<p><a href="https://docs.djangoproject.com/en/1.5/topics/http/sessions/">文档传送门</a>,
如果想让django项目支持session,则必须在settings.py中指定,见上.<strong>默认情况django使用 django.contrib.sessions.models.Session将session存储在你的数据库中.</strong>,这里我们查看下,如一个多用户的网站的django_session表:</p>

<pre><code>mysql&gt; select * from django_session;
+----------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
| session_key                      | session_data                                                                                                                                                                             | expire_date         |
+----------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
| 1x5fxr1upboyiw4ny640tgdl2mto6i6i | NjJkNzBiYjE2NjBlYTBkMGZmY2QzYmIxOGE5MDRmNTE1YTgzM2FmNzqAAn1xAS4=                                                                                                                         | 2014-07-16 02:21:35 |
..................
</code></pre>

<p>很长一大段,这里省略了.如果在settings.py的 INSTALLED_APPS中添加了<code>django.contrib.sessions</code>则运行<code>manage.py syncdb</code>将会生成数据库级别的session,表名为django-session.</p>

<p>为了提升性能,我们可以使用缓存级别的session,这点会在后面提及.</p>

<h2>1.在views中使用session</h2>

<p><strong>每个HttpRequest对象都有session属性</strong>,那么我们可以在views中使用:</p>

<pre><code> fav_color = request.session['fav_color']     #get
 request.session['fav_color'] = 'blue'           #set
 del request.session['fav_color']                  #del
 'fav_color' in request.session                     #contains
 fav_color = request.session.get('fav_color', 'red')
 fav_color = request.session.pop('fav_color')   # pop
</code></pre>

<p>session对象字典形式,有keys(),items(),setdefault(),clear()等方法,以下方法是常用的:</p>

<p>(1).<code>flush()</code></p>

<p>从session中删除数据,然后再生数据,比如django的logout()函数就会调用它.</p>

<p>(2).<code>set_test_cookie()</code></p>

<p>设置一个test cookie来判断用户浏览器是否接受cookie.</p>

<p>(3).<code>test_cookie_worked()</code></p>

<p>当设置了test cookie后返回True或Flase判断用户浏览器是否接受cookie.所以要先执行<code>set_test_cookie()</code>来下个套.</p>

<p>(4).<code>delete_test_cookie()</code></p>

<p>删除test cookie,测试完成后要收回自己下的套.</p>

<p>(5).<code>set_expiry(value)</code></p>

<p>设置过期时间,如果value是整数则表示N秒后过期,如果是时间或时间戳对象则表示指定到某一时间过期;如果value是0则在浏览器关闭后过期(会话session); 如果value 是None则使用全局session过期策略.</p>

<p>(6).<code>get_expiry_age()</code></p>

<p>返回session过期时间</p>

<pre><code>def login(request):
    if request.method == 'POST':
        if request.session.test_cookie_worked():    # 测套
            request.session.delete_test_cookie()      # 收套
            return HttpResponse("You're logged in.")
        else:
            return HttpResponse("Please enable cookies and try again.")
    request.session.set_test_cookie()             # 下套
    return render_to_response('foo/login_form.html')
</code></pre>

<h2>2.在模板中使用session</h2>

<p>如果在views设置了session,如<code>request.session['ms']</code>, 那么在模板中可以通过<code>{{ request.session.key }}</code>的形式使用:</p>

<pre><code>{{ request.session.ms }}
</code></pre>

<h2>3.在views外使用session</h2>

<p>从mysql检索出来的django_session表来看,包含字段如下:</p>

<pre><code>mysql&gt; describe django_session;
+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| session_key  | varchar(40) | NO   | PRI | NULL    |       |
| session_data | longtext    | NO   |     | NULL    |       |
| expire_date  | datetime    | NO   | MUL | NULL    |       |
+--------------+-------------+------+-----+---------+-------+
3 rows in set (0.01 sec)
</code></pre>

<p>那么可以使用django 数据库API来访问session,注意使用**get_decoded() **来读取实际的session数据,如下:</p>

<pre><code>&gt;&gt;&gt; from django.contrib.sessions.models import Session
&gt;&gt;&gt; s=Session.objects.all()
&gt;&gt;&gt; s
[&lt;Session: Session object&gt;, &lt;Session: Session object&gt;, &lt;Session: Session object&gt;, &lt;Session: Session object&gt;, &lt;Session: Session object&gt;,....]
&gt;&gt;&gt; for i in s:
...     print i.get_decoded()
... 
{'domain': 'http://beginman.sinaapp.com'}
{'domain': 'http://beginman.sinaapp.com', '_auth_user_backend': 'django.contrib.auth.backends.ModelBackend', '_auth_user_id': 1L}
{'domain': 'http://beginman.sinaapp.com'}
....
</code></pre>

<h2>4.其他细节</h2>

<p>ession 字典接受任何支持序列化的Python对象。 参考Python内建模块pickle的文档以获取更多信息。</p>

<p>Session 数据存在数据库表 django_session 中</p>

<p>Session 数据在需要的时候才会读取。 如果你从不使用 request.session ， Django不会动相关数据库表的一根毛。11</p>

<p>Django 只在需要的时候才送出cookie。 如果你压根儿就没有设置任何会话数据，它不会 送出会话cookie(除非 SESSION_SAVE_EVERY_REQUEST 设置为 True )。</p>

<p>Django session 框架完全而且只能基于cookie。 它不会后退到把会话ID编码在URL中（像某些工具(PHP,JSP)那样）。</p>

<p>这是一个有意而为之的设计。 把session放在URL中不只是难看，更重要的是这让你的站点 很容易受到攻击——通过 Referer header进行session ID”窃听”而实施的攻击。</p>

<h1>五.Session之存取Redis Django实现</h1>

<p>session可以在数据库级别,缓存级别,文件级别,cookie级别基础上存取,对于缓存级别而言无疑是最提升性能的,我们可以放在django缓存系统中,也可以在memcache中,也可以在Redis中. 这里比较推荐Redis.我们完全可以用Redis实现Session功能.</p>

<blockquote>
  <p>1.我们知道session其实是在cookie中保存了一个sessionid，用户每次访问都将sessionid发给服务器，服务器通过ID查找用户对应的状态数据。在这里我的处理方式也是在cookie中定义一个sessionid，程序需要取得用户状态时将sessionid做为key在Redis中查找。</p>
  
  <p>2.同时session支持用户在一定时间不访问将session回收。</p>
</blockquote>

<p>思路参考:http://www.cnblogs.com/ddyq/p/3151284.html</p>

<p>第三方库:<code>django-redis-sessions</code>,<a href="https://github.com/martinrusev/django-redis-sessions">github地址</a>,提供了Redis database backend for your sessions. 另一个第三方库<a href="https://github.com/sebleier/django-redis-cache">Redis Django Cache Backend</a> ,A cache backend for Django using the Redis datastructure server.用Redis存储缓存数据.</p>

<p>对此我们可以应用到自己的项目中,如上篇<a href="http://beginman.sinaapp.com/blog/75/">Redis key的设计模式 &amp; django登陆</a> 提到的实例,那么接下来我们在此基础上<strong>实现session,cookie,Redis定制的用户系统</strong>.</p>

<p><strong>注意要实现配置好Redis服务.</strong></p>

<h3>1.安装&amp;配置</h3>

<p><strong>(1).首先安装配置django-redis-sessions</strong>:</p>

<pre><code>pip install django-redis-sessions
</code></pre>

<p>在settings.py中设置<code>SESSION_ENGINE</code>,它默认是: django.contrib.sessions.backends.db 这里设置如下:</p>

<pre><code>SESSION_ENGINE = 'redis_sessions.session'
</code></pre>

<p>然后在settings.py中设置Redis数据库信息:</p>

<pre><code>SESSION_REDIS_HOST = 'localhost'
SESSION_REDIS_PORT = 6379
SESSION_REDIS_DB = 0
SESSION_REDIS_PASSWORD = 'password'
SESSION_REDIS_PREFIX = 'session'

# If you prefer domain socket connection, you can just add this line instead of SESSION_REDIS_HOST and SESSION_REDIS_PORT.

SESSION_REDIS_UNIX_DOMAIN_SOCKET_PATH = '/var/run/redis/redis.sock'
</code></pre>

<p>配置完成后可测试如下:</p>

<pre><code>$ pip install django nose redis
# Make sure you have redis running on localhost:6379
(poem)[beginman@beginman poem]$ nosetests

----------------------------------------------------------------------
Ran 0 tests in 0.001s

OK
</code></pre>

<p><strong>(2).安装配置django-redis-cache</strong></p>

<pre><code>pip install django-redis-cache 
</code></pre>

<p>然后在settings.py配置CACHES</p>

<p>#On Django &lt; 1.3:
    CACHE_BACKEND = 'redis_cache.cache://<host>:<port>'</p>

<pre><code>#On Django &gt;= 1.3:
# When using TCP connections
CACHES = {
    'default': {
        'BACKEND': 'redis_cache.RedisCache',
        'LOCATION': '&lt;host&gt;:&lt;port&gt;',
        'OPTIONS': {
            'DB': 1,
            'PASSWORD': 'yadayada',
            'PARSER_CLASS': 'redis.connection.HiredisParser',
            'CONNECTION_POOL_CLASS': 'redis.BlockingConnectionPool',
            'CONNECTION_POOL_CLASS_KWARGS': {
                'max_connections': 50,
                'timeout': 20,
            }
        },
    },
}

# When using unix domain sockets
# Note: ``LOCATION`` needs to be the same as the ``unixsocket`` setting
# in your redis.conf
CACHES = {
    'default': {
        'BACKEND': 'redis_cache.RedisCache',
        'LOCATION': '/path/to/socket/file',
        'OPTIONS': {
            'DB': 1,
            'PASSWORD': 'yadayada',
            'PARSER_CLASS': 'redis.connection.HiredisParser'
        },
    },
}

MIDDLEWARE_CLASSES = (
'django.middleware.cache.UpdateCacheMiddleware',    # This must be first on the list
'django.middleware.common.CommonMiddleware',
'django.contrib.sessions.middleware.SessionMiddleware',
(...)
'django.middleware.cache.FetchFromCacheMiddleware', # This must be last
</code></pre>

<p>注意关于TCP连接:</p>

<pre><code>#这里是TCP连接
CACHES = {
    'default': {
        'BACKEND': 'redis_cache.RedisCache',
        'LOCATION': '127.0.0.1:6379',
        'OPTIONS': {
            'DB': 0,
            'PASSWORD': '', # 这里没有设置密码
            # 'PARSER_CLASS': 'redis.connection.HiredisParser',  # 这段可先注释掉否则出现 :Hiredis is not installed的错误
            'CONNECTION_POOL_CLASS': 'redis.BlockingConnectionPool',
            'CONNECTION_POOL_CLASS_KWARGS': {
                'max_connections': 50,
                'timeout': 20,
            }
        },
    },
}
</code></pre>

<p>至此我们的Redis For Django算是完成了,如果不太明白,还有一篇文章<a href="http://michal.karzynski.pl/blog/2013/07/14/using-redis-as-django-session-store-and-cache-backend/"><strong>Using Redis as Django's session store and cache backend</strong></a>可以借鉴.</p>

<p>接下来我们可以在django中使用cache,存储在Redis中.如下:</p>

<pre><code># Start by importing your default cache:
from django.core.cache import cache

# Store data under a-unique-key:
cache.set('a-unique-key', 'this is a string which will be cached')

# Later on you can retrieve it in another function:
cache.get('a-unique-key') # Will return None if key is not found in cache

# You can specify a default value:
cache.get('another-unique-key', 'default value')

# You can store multiple values at once:
cache.set_many({'a': 1, 'b': 2, 'c': 3})

# And fetch multiple values:
cache.get_many(['a', 'b', 'c']) # returns {'a': 1, 'b': 2, 'c': 3}

# You can store complex types in the cache:
cache.set('a-unique-key', {
    'string'    : 'this is a string',
    'int'       : 42,
    'list'      : [1, 2, 3, 4],
    'tuple'     : (1, 2, 3, 4),
    'dict'      : {'A': 1, 'B' : 2},
})
</code></pre>

<p>如果我们选择socket连接方式而非TCP,那么在运行中可能会出现如下错误:Error 2 connecting to unix socket: /var/run/redis/redis.sock. No such file or directory.
这是因为redis 默认没有开启unix socket，需要在/etc/redis/redis.conf中修改,如下:</p>

<pre><code>unixsocket /var/run/redis/redis.sock
unixsocketperm 777
</code></pre>

<p>记住配置完成之后要重启Redis服务:</p>

<pre><code>$ sudo service redis-server restart
</code></pre>

<h1>六.Django+Redis实现用户系统</h1>

<p>在掌握上述知识后,我们可以使用django+redis来模拟django用户系统.接下来的例子那上篇博客开刀,项目放到github上了
<a href="https://github.com/BeginMan/RedisPoem">这里</a>. 上篇我们实现了登陆的功能,接下来要实现用户系统功能.</p>

<h2>1.思路</h2>

<p><img src="http://images.cnblogs.com/cnblogs_com/BeginMan/486940/o_redis_user_system.png" alt="" /></p>

<h2>2.程序实现</h2>

<h3>1.用户系统类</h3>

<p>这里模拟一个蹩脚的用户系统类(userSystem),如下:</p>

<pre><code>#coding=utf-8
#Redis实现用户系统
__author__ = 'beginman'
import redis
import datetime
import hashlib
r = redis.StrictRedis(host='localhost', port='6379', db=0)

class usSystem(object):
    def __init__(self, request,response=None, uid=0, **kwargs):
        self.request = request
        self.response = response
        self.kwargs = kwargs
        self.uid = uid          # user id
        self.sessionid = None

    def testCookie(self):
        """事先在登陆方法中下了request.session.set_test_cookie()的套子"""
        if self.request.session.test_cookie_worked():
            self.request.session.delete_test_cookie()
            return True
        return False

    def getUsObj(self):
        """返回用户对象,有则说明用户已登陆,无则注销"""
        self.sessionid = self.request.COOKIES.get('sessionid', None)
        if r.exists(self.sessionid):
            if r.exists('sessionid_%s' %self.sessionid):
                return r.hget('sessionid_%s' %self.sessionid, 'uid')
        return None

    def setCookieAndSession(self):
        """cookie在登陆成功后已经写入"""
        self.sessionid = self.request.COOKIES.get('sessionid', None)
        if not self.sessionid:
            # set cookie
            h = hashlib.md5()
            h.update(datetime.datetime.now())
            self.response.set_cookie('sessionid', h.hexdigest())
            self.sessionid = h.hexdigest()

        if not r.exists('sessionid_%s' %self.sessionid):
            #set session
            r.hset('sessionid_%s' %self.sessionid,'uid', self.uid)

        return True
</code></pre>

<p>然后在登陆方法中这样写:</p>

<pre><code># coding=utf-8
__author__ = 'beginman'
from django.shortcuts import render
from django.http import HttpResponseRedirect
from form import LoginForm
from common.userSystem import usSystem
import redis
import datetime
r = redis.StrictRedis(host='localhost', port='6379', db=0)


def home(request):
    return render(request, 'index.html')


def usLogin(request):
    context = {}
    if request.method == 'POST':
        form = LoginForm(request.POST)
        if form.is_valid():
            us = form.cleaned_data['us']
            pwd= form.cleaned_data['pwd']
            if r.exists('us:%s:id' %us):        # 检查是否存在该用户关系键值
                uid = r.get('us:%s:id' %us)     # 获取该用户在user表中对应的id
                if r.exists('user:%s' %uid):    # 检查是否存在该用户键值(如user:1)
                    us_, pwd_ = r.hmget('user:%s' %uid, 'username', 'pwd')  # 获取该用户的用户名密码
                    if us_ == us and pwd_ == pwd:   # 校验成功
                        r.hincrby('user:%s' %uid, 'login_count', 1)     # 登陆次数累加
                        r.hset('user:%s' %uid, 'last_login_date', datetime.datetime.now())  # 添加最近登陆
                        # set Cookies
                        res = HttpResponseRedirect('/')
                        ussys = usSystem(request, res, uid)
                        if ussys.testCookie() and ussys.setCookieAndSession():
                            return res

        context['msg'] = u'账号或密码错误'
        context['form'] = form

    request.session.set_test_cookie()
    form = LoginForm()
    context['form'] = form
    return render(request, 'login.html', context)
</code></pre>

<p>同时要注意中间件处理:</p>

<pre><code>#coding=utf-8
#中间件扩展
__author__ = 'beginman'
from django.http import HttpResponseRedirect
from django.conf import settings
from common.userSystem import usSystem

class Mymiddleware(object):
    def process_request(self, request):
        """Request预处理函数"""
        path = str(request.path)
        request.session['domain'] = settings.DOMAIN
        if path.startswith('/site_media/'):
            return None
        #验证登陆
        ussys = usSystem(request)
        if ussys.getUsObj():
            pass
</code></pre>

<p>在一些需要登陆后才能访问的可以写在验证登陆后面.这里还需要慢慢改进.</p>

<h1>参考</h1>

<p><a href="https://docs.djangoproject.com/en/1.5/topics/http/sessions/">1.How to use sessions</a></p>

<p><a href="http://djangobook.py3k.cn/2.0/chapter14/">2.第十四章： 会话、用户和注册</a></p>

<p><a href="http://redis-py.readthedocs.org/en/latest/">3.Redis-py 文档</a></p>

<p><a href="http://shen2.cn/2013/06/">4.用unix socket加速php-fpm、mysql、redis的连接</a></p>
