---
layout: post
title: "nodejs redis使用(一)"
description: "nodejs redis使用(一)"
category: "redis"
tags: [redis]
---
{% include JB/setup %}

<h2>一.基础</h2>

<p><a href="https://github.com/mranney/node_redis">redis for nodejs</a></p>

<p>可用安装纯js实现的redis客户端</p>

<pre><code>npm install redis
</code></pre>

<p>也可以安装C实现<code>hiredis</code>（<strong>快速且无阻塞</strong>）:</p>

<pre><code>npm install hiredis redis
</code></pre>

<p><strong>如果hiredis已经安装，则node_redis会默认使用它，否则就使用就是redis</strong></p>

<h2>二.认证</h2>

<p>大多数redis都需要加锁，使用的时候需要认证</p>

<pre><code>var redis = require("redis"),
    PORT = 6379,
    HOST = '127.0.0.1',
    PWD = 'yourpassword',
    DB = 0,
    OPTS = {auth_pass: PWD, selected_db:DB},
    client = redis.createClient(PORT, HOST, OPTS);

// redis 链接错误
client.on("error", function(error) {
    console.log(error);
});

//认证
client.auth(PWD,function(){
    console.log('Redis Auth Successful! &gt; redis_db:'+ DB);
});

//选择库
//如：选择库3进行操作
client.select('3', function(err){
   err ? console.log('选择库3失败:'+err)
       :
       (function(){
           console.log('已选择库3');
           //you code.
       })();
});
</code></pre>

<h2>三.使用</h2>

<pre><code>client.set("stringKey", "hello,world", redis.print);

/*
    1.可用数组代替多值, 如：
        client.set("some key", "some val");
        client.set(["some other key", "some val"]);
    2.callback可选
*/
client.mset(
    ['testKey 1', 'testVal 1', "testKey2", "testVal2"],
    function (err, res){
        console.log(err);       //null
        console.log(res);       //OK
    }
);

//同上
//client.mset('testKey 1', 'testVal 1', "testKey2", "testVal2", function (err, res){
//    console.log(err);
//    console.log(res);
//});

//如果key不存在则返回null
client.get("missingKey", function(err, reply) {
    // reply is null when the key is missing
    console.log(reply);
})
</code></pre>

<p><strong>注意：</strong></p>

<ol>
<li>命令不区分大小写</li>
<li>返回单行应答的redis命令，返回的JavaScript字符串，整数应答返回JavaScript的数字，“bulk”应答返回节点的缓冲器，而“多批量bulk”应答返回节点缓冲器的JavaScript数组。 HGETALL返回与缓冲区的哈希键键入对象。</li>
</ol>

<h2>四.API</h2>

<p><strong>客户机将发出关于连接到Redis的服务器的状态的一些事件</strong></p>

<h3>ready</h3>

<p>一旦redis连接就会发出<code>ready</code>事件，表示已经准备好接收命令，当这个事件触发之前client命令会存在队列中，当一切准备就绪后按顺序调用</p>

<pre><code>client.on('ready', function(err) {
    console.log('ready');
})
</code></pre>

<h3>connect</h3>

<p>Redis的Connection事件之一，在不设置client.options.no_ready_check的情况下，客户端触发connect同时它会发出ready，如果设置了client.options.no_ready_check，当这个stream被连接时会触发connect</p>

<h3>error</h3>

<p>当遇到一个错误连接Redis服务时，触发<code>error</code>事件</p>

<p><strong>需要注意的是“error”是在nodejs中是特殊事件类型。如果没有一个“error”事件侦听器，nodejs将退出。这通常是你想要的，但它可能会导致这样一些神秘的错误信息：</strong></p>

<pre><code>$ node example.js

node.js:50
    throw e;
    ^
Error: ECONNREFUSED, Connection refused
    at IOWatcher.callback (net:870:22)
    at node.js:607:9
</code></pre>

<p>如果node_redis异常的话，也会发出<code>error</code>事件</p>

<pre><code>client.on('error', function(err) {
   console.log('node redis异常：' + err);
});
</code></pre>

<h3>end</h3>

<p>当与redis服务的建立关闭时client发出<code>end</code>事件</p>

<pre><code>client.on('end', function(err) {
   console.log('end');
});
</code></pre>

<h4>drain</h4>

<p>当TCP连接redis服务后缓冲区可写后，会触发“drain”事件，这有助于你控制发送频率，你需要检查<code>client.command_queue.length</code>去决定减少你的发送频率,然后你就可以继续发送当你<code>drain</code>.</p>

<pre><code>client.on('drain', function(){
    console.log(client.command_queue);          //{ tail: [], head: [], offset: 0 }
    console.log(client.command_queue.length);   //0
})
</code></pre>

<h4>idle</h4>

<p>当没有等待命令时会发出<code>idle</code>事件。</p>

<h4>redis.createClient(port, host, options)</h4>

<p>默认port是6379，默认的host是127.0.0.1，options为一个对象，可以包含下列的属性：</p>

<ol>
<li><p>parser： redis协议的解析器，默认是hiredis，如果像我一样悲剧的装不上，那该项会被设置为javascript；</p></li>
<li><p>no_ready_check： 默认为false，当连接建立后，服务器端可能还处在从磁盘加载数据的loading阶段，这个时候服务器端是不会响应任何命令的，这个时候node_redis会发送INFO命令来检查服务器状态，一旦INFO命令收到响应了，则表明服务器端已经可以提供服务了，此时node_redis会触发“ready”事件，这就是为什么会有“connect”和“ready”事件之分，如果你关闭了这个功能，我觉得会出现一些貌似灵异的问题；</p></li>
<li><p>enable_offline_queue： 默认为true，前面提到过，当连接ready之前，client会把收到的命令放入队列中等待执行，如果关闭该项，所有ready前的调用都将会立刻得到一个error callback；</p></li>
<li><p>retry_max_delay： 默认为null，默认情况下client连接失败后会重试，每次重试的时间间隔会翻倍，直到永远，而设置这个值会增加一个阀值，单位为毫秒；</p></li>
<li><p>connect_timeout: 默认为false，默认情况下客户端将会一直尝试连接，设置该参数可以限制尝试连接的总时间，单位为毫秒；</p></li>
<li><p>max_attempts: 默认为null，可以设置该参数来限制尝试的总次数；</p></li>
<li><p>auth_pass： 默认为null，该参数用来认证身份。</p></li>
</ol>

<h3>client.auth(password, callback)</h3>

<p>这个接口用来认证身份的，如果你要连接的redis服务器是需要认证身份的，那么你必须确保这个方法是创建连接后第一个被调用的。需要注意的是，为了让使重连足够的简单，client.auth()保存了密码，用于每次重连后的认证，而回调只有会执行一次哦~~</p>

<p>另外你需要确保的是，千万别自作聪明的把client.auth()放在“ready”或“connect”事件的回调中，你将会得到一行神秘的报错：</p>

<p>Error: Ready check failed: ERR operation not permitted 只需要在redis.createClient()代码后直接调用clietn.auth()即可</p>

<h3>client.end()</h3>

<p>这个方法会强行关闭连接，并不会等待所有的响应。如果不想如此暴力，推荐使用client.quit()，该方法会在收到所有响应后发送QUIT命令。</p>

<pre><code>client.set("foo_rand000000000000", "some fantastic value");

client.get("foo_rand000000000000", function(err, reply) {
    console.log(reply.toString());
});
client.end();
</code></pre>

<p><strong>end对于超时情况下，一些被卡住或运行时间过长有或你想重新加载是十分有用的。</strong></p>

<h3>client.unref()</h3>

<p>这个方法作用于底层socket连接，可以在程序没有其他任务后自动退出，可以类比nodejs自己的unref()。这个方法目前是个试验特性，只支持一部分redis协议。</p>

<pre><code>/*
 Calling unref() will allow this program to exit immediately after the get command finishes.
 Otherwise the client would hang as long as the client-server connection is alive.
 */
client.unref();
client.get("foo", function (err, reply) {
    if (err) throw (err);
    console.log(reply);
});
</code></pre>

<h3>client.multi([commands])</h3>

<p>multi命令可以理解为打包，它会把命令都存放在队列里，直到调用exec方法，redis服务器端会一次性原子性的执行所有发来的命令，node_redis接口最终会返回一个Multi对象。</p>

<p>先以一个小例子进行：</p>

<pre><code>while (set_size &gt; 0) {
    client1.sadd('sKey', "member"+ set_size);
    set_size -- ;
}

// multi chain with an individual callback
client1.multi()
    .scard('sKey')                                          // 索引为0的命令元素，值为20
    .smembers('sKey')                                       // 索引为1的命令元素，值为member19,.....member1
    .keys('sKey*', function (err, replies)  {            // 索引为2的命令元素，值为sKey
        // NOTE: code in this callback is NOT atomic
        // this only happens after the the .exec call finishes.
        client1.mget(replies, redis.print);
})
    .dbsize()                                           // 索引为3的命令元素，值为59
    .exec(function (err, replies) {
        /*
            client.exec(callback)
            client.multi()返回一个Multi对象，
            Multi对象如客户端client一样共享redis命令，如上的.scard,.dbsize() .etc.
            命令以队列的形式排列在Multi对象里直到“Multi.exec()”方法被执行
        */
        console.log("MULTI got " + replies.length + ' replies: ' + replies + "\n");
        replies.forEach(function (reply, index) {
        console.log("reply " + index + ':' + reply.toString() + "\n");
    })
});
</code></pre>

<h3>Multi.exec( callback )</h3>

<p>The callback of <code>.exec()</code> will get invoked with two arguments:</p>

<ul>
<li><p><code>err</code> type: <code>null | Array</code> err is either null or an array of Error Objects corresponding the the sequence the commands where chained. The last item of the array will always be an <code>EXECABORT</code> type of error originating from the <code>.exec()</code> itself.</p></li>
<li><p><code>results</code> type: <code>null | Array</code> results is an array of responses corresponding the the sequence the commands where chained.</p></li>
</ul>

<p>You can either chain together <code>MULTI</code> commands as in the above example, or you can queue individual commands while still sending regular client command as in this example:</p>

<pre><code>var multi = client1.multi();            //将Multi对象赋值给multi变量
multi.incr("count", redis.print);       //将count自增, Reply: 101
multi.incr("count2", redis.print);      //将count2自增,Reply: 2

//立即执行(先执行)
client1.mset("count", 100, "count2", 1, redis.print);       //Reply: OK

//消费队列自动批量执行
multi.exec(function (err, replies) {
    console.log(replies);                //[ 101, 2 ]
});

//重新执行同一事务
multi.exec(function (err, replies) {
    console.log(replies);               //[ 102, 3 ]
    client1.quit();
});

//再重新执行同一事务
multi.exec(function (err, replies) {
    console.log(replies);                //[ 103, 4 ]
});
</code></pre>

<p>在终端执行情况如下：</p>

<pre><code>$ node test.js
Reply: OK
Reply: 101
Reply: 2
[ 101, 2 ]

Reply: 102
Reply: 3
[ 102, 3 ]

Reply: 103
Reply: 4
[ 103, 4 ]
</code></pre>

<p><strong>在MULTI对象队列中也可以使用命令数组，如下：</strong></p>

<pre><code>client1.multi([
    ["mget", "count", "count2", "misskeys", redis.print],
    ["incr", "count"],
    ["incr", "count2"]
]).exec(function (err, replies) {
    console.log(replies);
});
</code></pre>

<h3>Multi.exec(callback)</h3>

<p>client.multi()会返回一个Multi对象，它包含所有的命令，直到调用Multi.exec()。我们来主要说一下这个方法的回调函数中的两个参数：</p>

<ol>
<li><p>err: null或者Array，没有出错当然就会返回null，如果出错则返回命令队列链中对应的发生错误的命令的错误信息，数组中最后一个元素exec()方法自身的错误信息，这里我要说的是，可以看出这种方式所谓的原子性主要是指的命令链中的命令一定会保证一起在服务器端执行，而不是指的像关系型数据库那样的回滚功能；</p></li>
<li><p>results： null或者Array，返回命令链中每个命令的返回信息</p></li>
</ol>

<h2>友好的hash命令</h2>

<h3>client.hgetall(hash)</h3>

<p><code>HGETALL</code>命令返回js对象，如：</p>

<pre><code>client.hmset("Program", "Python", 1, "nodejs", 2, "java", 3, "objective-c", 4, redis.print);
client.hgetall("Program", function(err, reply) {
    console.dir(reply);         // { Python: '1', nodejs: '2', java: '3', 'objective-c': '4' }
    console.log(typeof (reply));    // object
})
</code></pre>

<h3>client.hmset(hash, obj, [callback])</h3>

<pre><code>client.hmset("myhashKey",                                           //hash键
            {"field1":"val1", "field2":"val2", "field3":"val3"},    //值
            function (err){                                         //回调函数
                console.log("client.hmset(hash, obj, [callback])");
            }
);
</code></pre>

<p>如下我们还可以这样来处理：</p>

<pre><code>client.hmset(hash, key1, val1, ... keyn, valn, [callback])
</code></pre>

<h2>发布和订阅(Publish / Subscribe)</h2>

<p>如下：</p>

<pre><code>/**
 * Created by fang on 15/3/26.
 */

var redis = require("redis"),
    PORT = 6379,
    HOST = '127.0.0.1',
    PWD = '2015yunlianxiQAZWSX',
    DB = 1,
    OPTS = {auth_pass: PWD, selected_db:DB},
    client1 = redis.createClient(PORT, HOST, OPTS),
    client2 = redis.createClient(PORT, HOST, OPTS),
    msg_count = 0;

client1.auth('2015yunlianxiQAZWSX');
client2.auth('2015yunlianxiQAZWSX');

//client1订阅
client1.on("subscribe", function(channel, count) {
    client2.publish("channelA", "I am sending a message.");
    client2.publish("channelA", "I am sending a second message.");
    client2.publish("channelA", "I am sending my last message.");
});

client1.on('message', function(channel, message) {
    console.log("client1 订阅频道 " + channel + " 发来的消息: " + message);
    msg_count ++;
    if (msg_count == 3) {
        client1.unsubscribe();
        client1.end();
        client2.end();
    }
});

client1.incr("did a thing");
client1.subscribe("channelA");

/*
 $ node test.js
 client1 订阅频道 channelA 发来的消息: I am sending a message.
 client1 订阅频道 channelA 发来的消息: I am sending a second message.
 client1 订阅频道 channelA 发来的消息: I am sending my last message.
 */
</code></pre>

<h2>参考</h2>

<p><a href="http://www.kazaff.me/2014/06/06/1513/">nodejs操作redis</a></p>
