---
layout: post
title: "tornado异步编程（1）基础"
description: "tornado异步编程（1）基础"
category: "tornado"
tags: [tornado]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<h2>1.理解阻塞非阻塞与同步异步的区别，</h2>

<p>下面摘自知乎：<a href="http://www.zhihu.com/question/19732473">地址：</a></p>

<blockquote>
  <p>1.<strong>阻塞非阻塞</strong>：可以简单理解为需要做一件事能不能立即得到返回应答，如果不能立即获得返回，需要等待，那就阻塞了，否则就可以理解为非阻塞。</p>
  
  <p>2.<strong>同步异步</strong>： 你总是做完一件再去做另一件，不管是否需要时间等待，这就是同步；异步呢则反之，你可以同时做几件事，并非一定需要一件事做完再做另一件事。同步简单理解成一问一答同步进行，异步可以简单理解为不必等一个问题有答了再去问另一个问题，尽管问，有答了再通知你。</p>
</blockquote>

<!--more-->

<blockquote>
  <p>3.举个例子： 
  我去买一本书，立即买到了，这就是非阻塞；
  如果恰好书店没有，我就等一直等到书店有了这本书买到了才走，这就是阻塞；
  如果书店恰好没有，我就告诉书店老板，书来了告诉我一声让我来取或者直接送到我家，然后我就走了，这就是异步。
  那同步呢？ 前面两种情况，非阻塞和阻塞都可以称为同步。
  如果说书店有这书，我还让老板通知我以后来取就没这个必要了。</p>
</blockquote>

<p>反映在编程方面就是 用户进程 调用 系统调用。(用户进程对应我，内核 对应 书店老板，书对应数据资源data ， 买书就是一个系统调用了)这阻塞非阻塞与同步异步IO机制，都是伴随计算机系统发展，用来解决一些出现的问题。阻塞非阻塞、同步异步可以组合，但是没必要组合，应该说是不同的IO机制，没必要纠结怎么区分，如果定要组合心里才爽，可以 这样认为：阻塞非阻塞都是同步，异步就没什么阻塞不阻塞了，都异步了还阻塞啥，肯定是非阻塞了。(异步非阻塞听起来多别扭)</p>

<p>unix网络编程中说到：</p>

<p>将IO模型分为五类：阻塞IO，非阻塞IO，IO复用，信号驱动，异步IO</p>

<p>其中阻塞IO就是那种recv, read，一直等，等到有了拷贝了数据才返回；</p>

<p>非阻塞就是不用等，立即返回，设置描述符为非阻塞就行了，但是要进程自己一直检查是否可读；</p>

<p>IO复用其实也是阻塞的，不过可以用来等很多描述符；</p>

<p>信号驱动采用信号机制等待；</p>

<p>异步IO就不用等待了，当他告知你的时候，已经可以返回了，数据都拷贝好了。</p>

<p>posix.1严格定义的异步IO是要求没有任何一点阻塞，而上述的前面四个（阻塞IO，非阻塞IO，IO复用，信号驱动）都不同程度阻塞了，而且都有一个共同的阻塞： 内核拷贝数据到进程空间的这段时间需要等待。 (这么说上面的举例中： 必须要书送到我家，否则都不算异步，纠结。。。)</p>

<p>也可参考这里：<a href="http://www.brucesky.com/articles/896">深入可伸缩I/O模型：同步和异步方式</a></p>

<h2>2.tornado异步编程</h2>

<p>如下例子：</p>

<pre><code>class MainHandler2(tornado.web.RequestHandler):
def get(self, *args, **kwargs):
    http = tornado.httpclient.HTTPClient()
    response = http.fetch("http://friendfeed-api.com/v2/feed/bret")
    if response.error:
        raise tornado.web.HTTPError(500)
    json = tornado.escape.json_decode(response.body)
    self.write(json)
</code></pre>

<p>tornado server的整体性能依赖于访问google的时间，如果访问google的时间比较长，就会导致整体server的阻塞,比如说这个请求要花费5s.</p>

<p>(1)如果这些代码是单线程运行，在这5秒钟的时间里，服务器不能干其他任何事情，所以，你的服务效率是每秒0.2个请求,服务器接受客户端请求的次数： 1/5= 0.2次/s</p>

<p>(2)如果开启多线程，比如20个，则性能会提高20倍，请求次数： 20/5 = 4次/s，  但是不断提高线程数量来解决问题是不够的，线程的内存和调度方面的开销是很昂贵的。</p>

<p>(3)异步IO (asynchronous IO  AIO)，</p>

<pre><code>class MainHandler(tornado.web.RequestHandler):
@tornado.web.asynchronous
def get(self):
    http = tornado.httpclient.AsyncHTTPClient()
    http.fetch("http://friendfeed-api.com/v2/feed/bret",
               callback=self.on_response)

def on_response(self, response):
    if response.error: raise tornado.web.HTTPError(500)
    json = tornado.escape.json_decode(response.body)
    self.write(json)
    self.finish()
</code></pre>

<p>然后我们做压力测试：</p>

<pre><code>#首先对同步测压
siege http://localhost:8123/ -c30 -t20s
...
#结果执行了7次就断掉了，测压结果：
Transactions:                  7 hits
Availability:             100.00 %
Elapsed time:              19.72 secs
Data transferred:           2.71 MB
Response time:             10.94 secs
Transaction rate:           0.35 trans/sec
Throughput:             0.14 MB/sec
Concurrency:                3.88
Successful transactions:           7
Failed transactions:               0
Longest transaction:           18.90
Shortest transaction:           0.00

#然后对异步进行测压
#基本上全部执行成功
Transactions:                 68 hits
Availability:             100.00 %
Elapsed time:              19.94 secs
Data transferred:           0.00 MB
Response time:              6.54 secs
Transaction rate:           3.41 trans/sec
Throughput:             0.00 MB/sec
Concurrency:               22.29
Successful transactions:          68
Failed transactions:               0
Longest transaction:           11.09
Shortest transaction:           1.98
</code></pre>

<p>可以明显看到异步的高性能。</p>

<p>上面的例子，主要有几个变化：</p>

<p>使用asynchronous decorator，它主要设置_auto_finish为false，这样handler的get函数返回的时候tornado就不会关闭与client的连接。</p>

<p>使用AsyncHttpClient，fetch的时候提供callback函数，这样当fetch http请求完成的时候才会去调用callback，而不会阻塞。</p>

<p>callback调用完成之后通过finish结束与client的连接。</p>

<p><img src="http://dl2.iteye.com/upload/attachment/0043/2696/e71c128e-a828-3aa6-9f59-bd40d60063f2.jpg" alt="" /></p>

<h2>3.tornado.gen</h2>

<p>Tornado 通过 @asynchronous decorator 来实现异步请求，但使用的时候必须将 request handler 和 callback 分离开，tornado.gen 模块可以帮助我们在一个函数里完成这两个工作。</p>

<p>还是上面的例子，接下来用gen实现：</p>

<pre><code>class GenMainHandler(tornado.web.RequestHandler):
#gen
@tornado.web.asynchronous
@tornado.gen.engine
def get(self, *args, **kwargs):
    http = tornado.httpclient.AsyncHTTPClient()

    response = yield tornado.gen.Task(http.fetch, 'http://friendfeed-api.com/v2/feed/bret')

    if response.error:
        raise tornado.web.HTTPError(500)

    json = tornado.escape.json_decode(response.body)
    self.write(json)
    self.finish()
</code></pre>

<p>关于<code>tornado.gen</code>,可参考下面两篇文章：</p>

<p><a href="http://www.zouyesheng.com/generator-for-async.html"><strong>1.使用生成器展平异步回调结构</strong></a></p>

<p><a href="http://blog.xiaogaozi.org/2012/09/21/understanding-tornado-dot-gen/"><strong>2.理解 tornado.gen</strong></a></p>

<p>yield演示地址：</p>

<p><a href="http://pythontutor.com/visualize.html#code=S+%3D+%5B'EMPTY'%5D%0Adef+gen()%3A%0A++++for+i+in+%5B'a',+'b'%5D%3A%0A++++++++print+i%0A%0A++++for+i+in+range(10)%3A%0A++++++++S%5B0%5D+%3D+yield+i%0A++++++++print+str(S%5B0%5D),+'index%3A%25s'+%25+i%0A%0Ag+%3D+gen()%0Aprint+g.next()%0Aprint+g.send('ok')%0Aprint+g.next()&amp;mode=display&amp;origin=opt-frontend.js&amp;cumulative=false&amp;heapPrimitives=false&amp;drawParentPointers=false&amp;textReferences=false&amp;showOnlyOutputs=false&amp;py=2&amp;rawInputLstJSON=%5B%5D&amp;curInstr=0">yield原理演示：</a></p>
