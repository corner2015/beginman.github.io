---
layout: post
title: "Redis 一小时入门"
description: "Redis 一小时入门"
category: "redis"
tags: [redis]
---
{% include JB/setup %}

<h1>一.Redis身份介绍</h1>

<p>Redis是REmote DIctionary Server(Redis) 的缩写,属于NoSQL,利用key-value存储系统。redis的出现，很大程度补偿了memcached([mem'kæʃt])这类keyvalue存储的不足,同时提供了Python,Ruby,PHP等客户端.</p>

<!--more-->

<h1>二.Redis的核心原理</h1>

<p>我有这样一个习惯,在接触一门新技术之前首先要弄清楚以下几点:</p>

<p>1.<code>是什么?</code></p>

<p>2.<code>什么用?</code></p>

<p>3.<code>核心原理是什么?</code></p>

<p>4.<code>如何用?</code></p>

<p>5.<code>循序渐进的学</code></p>

<p>对于Redis我将会按照上述5个步骤进行粗略的学习.关于第1,2在上节<a href="http://beginman.sinaapp.com/blog/67/">Redis学习(1):初步认知,安装,部署与应用</a>已经提到了,接下来就要弄明白它的核心原理,更深层次的剖析将会在后期的学习总结中萌生.</p>

<p>在<a href="http://blog.sina.com.cn/s/blog_5a15b7d10101gizu.html">Redis原理介绍</a>一文中总结如下:</p>

<blockquote>
  <p>1.key value store.是一个以key-value形式存储的数据库，定位直指MySQL，用来作为唯一的存储系统。</p>
  
  <p>2.memory cache.是一个把数据存储在内存中的高速缓存，用来在应用和数据库间提供缓冲，替代memcachd。
  3.data structrue server.把它支持对复杂数据结构的高速操作作为卖点，提供某些特殊业务场景的计算和展现需求。比如排行榜应用，Top 10之类的。</p>
  
  <p>目前更多的人还是把它定位为一个memcached的升级版，提供更多的数据结构操作，仍然是一个cache。</p>
</blockquote>

<p><strong>所有用关系型数据库理论可以描述的结构，用Redis的数据结构，都可以实现。</strong></p>

<h1>三.Redis的优点</h1>

<p>在前文也说过,在<a href="http://oldblog.antirez.com/post/take-advantage-of-redis-adding-it-to-your-stack.html"><strong>How to take advantage of Redis just adding it to your stack</strong></a>中作者说:</p>

<blockquote>
  <p>Redis is different than other database solutions(解决方案) in many ways: it uses memory as main storage support and disk only for persistence(持续), the data model is pretty unique(独一无二的人或物), it is single threaded and so forth(向前). I think that another big difference is that in order to take advantage of Redis in your production environment you don't need to switch to Redis. You can just use it in order to do new things that were not possible before, or in order to fix old problems.</p>
</blockquote>

<p>这是一篇好文,抽空翻译下.</p>

<h1>四.应用场景</h1>

<p>1.取最新N个数据的操作</p>

<p>2.排行榜应用，取TOP N操作</p>

<p>3.高速缓存</p>

<p>4.并发量大的应用程序</p>

<p>5.实时动态程序</p>

<p>等等...</p>

<p>在<a href="http://blog.nosqlfan.com/html/2235.html"><strong>Redis作者谈Redis应用场景</strong></a>中可参考更多</p>

<p>给大家推荐一篇好文<a href="http://highscalability.com/blog/2011/7/6/11-common-web-use-cases-solved-in-redis.html"><strong>11 Common Web Use Cases Solved In Redis</strong></a>,这个抽空也可以翻译下.</p>

<p>张善友的<a href="http://www.cnblogs.com/shanyou/archive/2012/09/04/2670972.html"><strong>Redis应用场景</strong></a>则是从每个数据类型讲解其应用场景.</p>

<h1>五.谁在用</h1>

<p>Twitter  Github  Weibo  Pinterest  Snapchat  Craigslist  Digg  StackOverflow  Flickr等巨头都在用,关于谁在用,我特地Google一篇文章<a href="http://www.csdn.net/article/2013-10-07/2817107-three-giant-share-redis-experience"><strong>国内外三个不同领域巨头分享的Redis实战经验及使用场景</strong></a>,可惜造诣太浅,理解起来有些困难,且当了解,望众大牛云云.</p>

<h1>六.数据类型</h1>

<p>Redis常用的数据类型和对应的命令如下思维导图:<a href="http://naotu.baidu.com/?shareId=3ufwmthx8cu">Mind</a></p>

<p><img src="http://images.cnblogs.com/cnblogs_com/BeginMan/486940/o_data2.png" alt="" /></p>

<h2>1.String类型</h2>

<h3>(1)Redis能存储二进制安全的字符串，最大长度为1GB</h3>

<pre><code>127.0.0.1:6379&gt; set name "BeginMan"
OK
127.0.0.1:6379&gt; get name
"BeginMan"
127.0.0.1:6379&gt; get names
(nil)
</code></pre>

<p><code>**set**</code>:<a href="http://www.redis.cn/commands/set.html">语法</a></p>

<blockquote>
  <p>SET key value, 创建key并赋value,若key存在则被覆盖.状态码：总是OK，因为SET不会失败。</p>
</blockquote>

<p><code>**get**</code>:<a href="http://www.redis.cn/commands/get.html">语法</a></p>

<blockquote>
  <p>返回key的value。如果key不存在，返回特殊值nil。如果key的value不是string，就返回错误，因为GET只处理string类型的values。</p>
</blockquote>

<h3>(2).批量的读写操作</h3>

<pre><code>127.0.0.1:6379&gt; mset name "beginman" age 23 sex "male"
OK
127.0.0.1:6379&gt; mget name age sex
1) "beginman"
2) "30"
3) "male"
127.0.0.1:6379&gt; get name
"beginman"
</code></pre>

<p><code>**mset**</code>,<a href="http://www.redis.cn/commands/mset.html">语法</a></p>

<blockquote>
  <p>MSET key value [key value ...],相当于多个set的操作</p>
</blockquote>

<p><code>**mget**</code>,<a href="http://www.redis.cn/commands/mget.html">语法</a></p>

<blockquote>
  <p>MGET key [key ...],相当于多个get的操作</p>
</blockquote>

<h3>(3).数值运算,支持加减运算</h3>

<pre><code>127.0.0.1:6379&gt; incr age
(integer) 24
127.0.0.1:6379&gt; incrby age 2
(integer) 26
127.0.0.1:6379&gt; incrby age -2
(integer) 24
127.0.0.1:6379&gt; decr age
(integer) 23
127.0.0.1:6379&gt; decrby age 3
(integer) 20
127.0.0.1:6379&gt; incr ages   # ages原本不存在
(integer) 1
127.0.0.1:6379&gt; get ages
"1"
</code></pre>

<p><code>**incr**</code>,<a href="http://www.redis.cn/commands/incr.html">语法</a></p>

<blockquote>
  <p>INCR key, 对key对应的数字做加1操作。如果key不存在则自动创建并赋值为0,然后在+1.如果对应value不是能表示成整数的字符串则会报错.Redis没有整数类型,会自动将String类型的数值转换成整数.</p>
</blockquote>

<p><code>**incrby**</code>,<a href="http://www.redis.cn/commands/incrby.html">语法</a></p>

<blockquote>
  <p>INCRBY key increment,将对应value加increment. 机制同incr.</p>
</blockquote>

<p><code>**decr**</code>和<code>**decrby**</code>机制同它们两个,只不过作用相反而已.不在累述.</p>

<h3>(4).修改,截取</h3>

<p><code>**append**</code>,<a href="http://www.redis.cn/commands/append.html">语法</a></p>

<blockquote>
  <p>APPEND key value,如果 key 已经存在，并且值为字符串，那么这个命令会把 value 追加到原来值（value）的结尾。 如果 key 不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况 APPEND 将类似于 SET 操作。</p>
</blockquote>

<pre><code>127.0.0.1:6379&gt; append name ".hello"
(integer) 14
127.0.0.1:6379&gt; get name
"BeginMan.hello"
127.0.0.1:6379&gt; append iexistsKey "Jack"
(integer) 4
127.0.0.1:6379&gt; get iexistsKey
"Jack"
</code></pre>

<p><code>**strlen**</code>,<a href="">语法</a></p>

<blockquote>
  <p>STRLEN key,返回key的string类型value的长度。如果key对应的非string类型，就返回错误。如果不存在key则返回0</p>
</blockquote>

<pre><code>127.0.0.1:6379&gt; strlen name
(integer) 14
127.0.0.1:6379&gt; strlen jack
(integer) 0
</code></pre>

<p><code>substr</code>:</p>

<blockquote>
  <p>substr 返回截取过的key的字符串值,注意并不修改key的值。下标是从0开始的.(redis在2.0版本以后不包括2.0,使用的方法是getrange 参数相同.</p>
</blockquote>

<p>127.0.0.1:6379> substr name 0 4
"Begin"</p>

<p><code>**getrange**</code>,<a href="http://www.redis.cn/commands/getrange.html">语法</a></p>

<blockquote>
  <p>GETRANGE key start end.这个命令是被改成GETRANGE的，在小于2.0的Redis版本中叫SUBSTR。 返回key对应的字符串value的子串，这个子串是由start和end位移决定的（两者都在string内）。可以用负的位移来表示从string尾部开始数的下标。所以-1就是最后一个字符，-2就是倒数第二个，以此类推。
  这个函数处理超出范围的请求时，都把结果限制在string内。</p>
</blockquote>

<pre><code>127.0.0.1:6379&gt; getrange name 0 4
"Begin"
127.0.0.1:6379&gt; getrange name 0 400000
"BeginMan.hello"
127.0.0.1:6379&gt; getrange name 0 -1
"BeginMan.hello"
127.0.0.1:6379&gt; getrange name -3 -1
"llo"
</code></pre>

<h2>2. List类型</h2>

<p>在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List中可以包含的最大元素数量是4294967295。</p>

<pre><code>127.0.0.1:6379&gt; lpush mykey a b c d 
(integer) 4
127.0.0.1:6379&gt; lrange mykey 0 2
1) "d"
2) "c"
3) "b"
127.0.0.1:6379&gt; lrange mykey 0 -1
1) "d"
2) "c"
3) "b"
4) "a"

127.0.0.1:6379&gt; lpushx mykey2 e
(integer) 0
127.0.0.1:6379&gt; lpushx mykey e
(integer) 5
127.0.0.1:6379&gt; lpushx mykey c
(integer) 6
127.0.0.1:6379&gt; lrange mykey 0 0
1) "c"
127.0.0.1:6379&gt; lpush mykey 1
(integer) 7
127.0.0.1:6379&gt; rpush mykey 0
(integer) 8
127.0.0.1:6379&gt; lrange mykey 0 -1
1) "1"
2) "c"
3) "e"
4) "d"
5) "c"
6) "b"
7) "a"
8) "0"
127.0.0.1:6379&gt; 
</code></pre>

<p>更多内容参考<a href="http://www.cnblogs.com/stephen-liu74/archive/2012/02/14/2351859.html"><strong>Redis学习手册(List数据类型)</strong></a></p>

<h2>3. set类型</h2>

<p>set同list,能够对它增加,删除,查找,判断等,set是无序的集合.还可实现交集，并集，差集等.</p>

<pre><code>127.0.0.1:6379&gt; sadd myset a b c          # 由于该键myset之前并不存在，因此参数中的三个成员都被正常插入
(integer) 3
127.0.0.1:6379&gt; smembers myset            # 查看set
1) "c"
2) "a"
3) "b"
127.0.0.1:6379&gt; sadd myset a c d e        # 由于 a,c已经存在所有只插入了d e
(integer) 2
127.0.0.1:6379&gt; smembers myset
1) "d"
2) "c"
3) "a"
4) "b"
5) "e"
127.0.0.1:6379&gt; 

127.0.0.1:6379&gt; sismember myset a      #判断a是否已经存在，返回值为1表示存在。否则返回0
(integer) 1

127.0.0.1:6379&gt; scard myset            # 获取集合元素数量
(integer) 5


127.0.0.1:6379&gt; srandmember myset     # 随即返回一个元素
"e"


127.0.0.1:6379&gt; spop myset            # 随机的移除并返回Set中的某一成员
"d"

127.0.0.1:6379&gt; srem myset a b c      #删除参数中指定的成员，不存在的参数成员将被忽略，如果该Key并不存在，将视为空Set处理
(integer) 3

127.0.0.1:6379&gt; smembers myset
1) "e"
127.0.0.1:6379&gt; smove myset myset2 e    #将e从myset移到myset2
(integer) 1
127.0.0.1:6379&gt; smembers myset
(empty list or set)

#myset和myset2相比，a、b和d三个成员是两者之间的差异成员。再用这个结果继续和myset3进行差异比较，b和d是myset3不存在的成员。
redis 127.0.0.1:6379&gt; sdiff myset myset2 myset3
1) "d"
2) "b"

# 交集
127.0.0.1:6379&gt; sinter myset myset2
1) "e"
#并集
127.0.0.1:6379&gt; sunion myset myset2
1) "e"
2) "a"
3) "c"
4) "b"
127.0.0.1:6379&gt; 
</code></pre>

<h2>4.有序集合（Sorted Sets）类型</h2>

<p>Sorted Sets和Sets结构相似，不同的是存在Sorted Sets中的数据会有一个score属性，并会在写入时就按这个score排好序。</p>

<pre><code>#添加一个分数为1,2,3的成员。
127.0.0.1:6379&gt; zadd myzset 1 "one" 2 "two" 3 "three"
(integer) 3

#0表示第一个成员，-1表示最后一个成员。WITHSCORES选项表示返回的结果中包含每个成员及其分数，否则只返回成员。
127.0.0.1:6379&gt; zrange myzset 0 -1 withscores
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"

127.0.0.1:6379&gt; zrange myzset 0 -1
1) "one"
2) "two"
3) "three"


#获取成员one在Sorted-Set中的位置索引值。0表示第一个位置。
redis 127.0.0.1:6379&gt; zrank myzset one
(integer) 0

#成员four并不存在，因此返回nil。
redis 127.0.0.1:6379&gt; zrank myzset four
(nil)

#获取myzset键中成员的数量。    
redis 127.0.0.1:6379&gt; zcard myzset
(integer) 3

#返回与myzset关联的Sorted-Set中，分数满足表达式1 &lt;= score &lt;= 2的成员的数量。
redis 127.0.0.1:6379&gt; zcount myzset 1 2
(integer) 2

#删除成员one和two，返回实际删除成员的数量。
redis 127.0.0.1:6379&gt; zrem myzset one two
(integer) 2 

#获取成员three的分数。返回值是字符串形式。
redis 127.0.0.1:6379&gt; zscore myzset three
"3"  

#将成员one的分数增加100，并返回该成员更新后的分数。
127.0.0.1:6379&gt; zincrby myzset 100 one
"100"

127.0.0.1:6379&gt; zrange myzset 0 -1 withscores
1) "three"
2) "3"
3) "one"
4) "100"
</code></pre>

<h2>5.Hash类型</h2>

<p>Redis能够存储key对多个属性的数据（比如user1.uname user1.passwd）</p>

<pre><code># (h set)给键值为myhash的键设置字段为field1，值为Jack
127.0.0.1:6379&gt; hset myhash field1 "Jack"
(integer) 1

# (h get)获取键值
127.0.0.1:6379&gt; hget myhash field1
"Jack"
127.0.0.1:6379&gt; hget myhash field2
(nil)

# hash长度(字段数量。)
127.0.0.1:6379&gt; hlen myhash
(integer) 1

# 判断字段是否存在 有则返回1,无则0
127.0.0.1:6379&gt; hexists myhash field1
(integer) 1

#hdel删除字段删除成功返回1。
127.0.0.1:6379&gt; hdel myhash field1
(integer) 1

#通过hsetnx命令给myhash添加新字段field1，其值为stephen，因为该字段已经被删除，所以该命令添加成功并返回1
127.0.0.1:6379&gt; hsetnx myhash field1 BeginMan
(integer) 1
127.0.0.1:6379&gt; hget myhash field1
"BeginMan"
127.0.0.1:6379&gt; 
</code></pre>

<h1>七.数据存储与过期</h1>

<p>Redis数据可以存储在内存,磁盘,log文件中,默认的是过期时间是-1,表示永不过期,可以通过<code>TTL</code>查看过期时间,通过<code>expire</code>设置过期时间.</p>

<pre><code>127.0.0.1:6379&gt; ttl name
(integer) -1                      # 永不过期
127.0.0.1:6379&gt; exists name       # 判断是否存在
(integer) 1
127.0.0.1:6379&gt; expire name 3     # 设置过期时间3秒
(integer) 1

127.0.0.1:6379&gt; exists name       # 到了过期时间自动删除 
(integer) 0
</code></pre>

<p>可以使用<code>expireat</code> 设置时间戳.EXPIREAT 的作用和 EXPIRE类似，都用于为 key 设置生存时间。不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳 Unix timestamp 。</p>

<pre><code>EXPIREAT mykey 1406021665         # 设置过期时间为 2014/7/22 17:34:25
</code></pre>

<h1>八.Redis DB</h1>

<p>Redis支持多个DB，默认是16个，你可以设置将数据存在哪一个DB中，不同DB间的数据具有隔离性。也可以在多个DB间移动数据。</p>

<pre><code>redis 127.0.0.1:6379&gt; SELECT 0
OK
redis 127.0.0.1:6379&gt; SET name "John Doe"
OK
redis 127.0.0.1:6379&gt; SELECT 1
OK
redis 127.0.0.1:6379[1]&gt; GET name
(nil)
redis 127.0.0.1:6379[1]&gt; SELECT 0
OK
redis 127.0.0.1:6379&gt; MOVE name 1
(integer) 1
redis 127.0.0.1:6379&gt; SELECT 1
OK
redis 127.0.0.1:6379[1]&gt; GET name
"John Doe"
</code></pre>

<p>Redis还能进行一些如下操作，获取一些运行信息</p>

<pre><code>redis 127.0.0.1:6379[1]&gt; DBSIZE
(integer) 1
redis 127.0.0.1:6379[1]&gt; INFO
redis_version:2.2.13
redis_git_sha1:00000000
redis_git_dirty:0
arch_bits:64
multiplexing_api:kqueue
......
</code></pre>

<p>Redis还支持对某个DB数据进行清除（当然清空所有数据的操作也是支持的）</p>

<pre><code>redis 127.0.0.1:6379&gt; SET name "John Doe"
OK
redis 127.0.0.1:6379&gt; DBSIZE
(integer) 1
redis 127.0.0.1:6379&gt; SELECT 1
OK
redis 127.0.0.1:6379[1]&gt; SET name "Sheldon Cooper"
OK
redis 127.0.0.1:6379[1]&gt; DBSIZE
(integer) 1
redis 127.0.0.1:6379[1]&gt; SELECT 0
OK
redis 127.0.0.1:6379&gt; FLUSHDB
OK
redis 127.0.0.1:6379&gt; DBSIZE
(integer) 0
redis 127.0.0.1:6379&gt; SELECT 1
OK
redis 127.0.0.1:6379[1]&gt; DBSIZE
(integer) 1
redis 127.0.0.1:6379[1]&gt; FLUSHALL
OK
redis 127.0.0.1:6379[1]&gt; DBSIZE
(integer) 0
</code></pre>

<h1>九.Redis的Python客户端</h1>

<p><a href="http://www.redis.cn/clients.html">Redis的客户端</a> 一应俱全,<code>redis-py</code>较为推荐.可参考<a href="http://hao.jobbole.com/python-redis-py/">redis-py：Redis数据库的Python客户端程序</a></p>

<p>安装:</p>

<pre><code>[beginman@beginman ~]$ sudo pip install redis
</code></pre>

<p>使用:</p>

<pre><code>&gt;&gt;&gt; import redis
&gt;&gt;&gt; r = redis.StrictRedis(host='localhost', port='6379', db=0)
&gt;&gt;&gt; r.set('name', "Beginman")
True
&gt;&gt;&gt; r.get('name')
'Beginman'
</code></pre>

<h1>十.参考</h1>

<p><a href="http://blog.nosqlfan.com/html/3139.html?ref=rediszt">1.Redis系统性介绍</a></p>

<p><a href="http://www.cnblogs.com/stephen-liu74/category/354125.html">2.Redis学习手册</a></p>

<p><a href="http://blog.sina.com.cn/s/blog_5a15b7d10101gizu.html">3.Redis原理介绍 </a></p>

<h1>十一.推荐学习:</h1>

<p>题为1小时入门Redis,完成这篇博客实则一下午的时间,对于入门可谓"管中窥豹可见一斑",还需深入学习,系统学习.</p>

<p><a href="http://www.cnblogs.com/stephen-liu74/category/354125.html">1.<strong>Redis学习手册</strong></a>[强烈推荐]</p>

<p><a href="http://redisbook.readthedocs.org/en/latest/index.html">2.<strong>Redis 设计与实现（第一版）</strong></a>[强烈推荐]</p>
