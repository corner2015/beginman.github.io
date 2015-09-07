---
layout: post
title: "Objective-C基础(1)"
description: "Objective-C基础(1)"
category: "Objective-c"
tags: [Objective-c]
---
{% include JB/setup %}

<h1>一.零碎总结</h1>

<ul>
<li><code>.m</code>是Object-C的扩展名，代表message</li>
<li>Object-C中<code>#import</code>代替了C的<code>#include</code></li>
<li>NS前缀，CoCoa给所有函数，常量和类型名称都添加了NS前缀，表示其属于CoCoa阵营的，除了宣誓主权外，还有避免名称冲突的作用。</li>
</ul>

<!--more-->

<h1>二.NSLog</h1>

<p>NSLog定义在NSObjCRuntime.h中，如下所示：</p>

<pre><code>void NSLog(NSString *format, …);
</code></pre>

<p>基本上，NSLog很像printf，同样会在console中输出显示结果。不同的是，** 传递进去的格式化字符是NSString的对象，而不是<code>chat *</code>这种字符串指针。**</p>

<p><strong>示例</strong></p>

<p>[c] NSLog (@"this is a test"); NSLog (@"string is :%@", string); NSLog (@"x=%d, y=%d", 10, 20); [/c]</p>

<p>但是下面的写法是不行的：</p>

<pre><code>int i = 12345;
NSLog( @"%@", i );
</code></pre>

<p>原因是， <strong>%@需要显示对象</strong>，而int i明显不是一个对象</p>

<p><strong>格式</strong></p>

<p>NSLog的格式如下所示：</p>

<ul>
<li>%@ 对象</li>
<li>%d, %i 整数</li>
<li>%u 无符整形</li>
<li>%f 浮点/双字</li>
<li>%x, %X 二进制整数</li>
<li>%o 八进制整数</li>
<li>%zu size_t</li>
<li>%p 指针</li>
<li>%e 浮点/双字 （科学计算）</li>
<li>%g 浮点/双字 </li>
<li>%s C 字符串</li>
<li>%.*s Pascal字符串</li>
<li>%c 字符</li>
<li>%C unichar</li>
<li>%lld 64位长整数（long long）</li>
<li>%llu 无符64位长整数</li>
<li>%Lf 64位双字</li>
</ul>

<h1>三.布尔类型</h1>

<p>Object-C的布尔类型不同于C，用<code>YES</code>,<code>NO</code>表示。</p>

<pre><code>[c] 
# import &lt;Foundation/Foundation.h&gt; //查找Foundation库的Foundation.h头文件 
BOOL areIntsDiffent(int thing1, int thing2) 
{ BOOL result = thing1 == thing2 ? NO : YES; return (result); } 
//boolstring返回值类型是指向NSString的指针,如下返回一个Cocoa字符串 
NSString *boolstring(BOOL yesNo) { return (yesNo == NO ? @"NO" : @"YES"); }

int main(int argc, const char *argv[])
{
    NSLog(@"hello world");              //@是本体
    NSLog(@"%d",areIntsDiffent(20, 40));    // 1
    NSLog(@"%@", boolstring(NO));           //NO

    NSLog(@"%d", areIntsDiffent(20, 40) == YES);//1
    return (0);
}//main
[/c]
</code></pre>

<h1>参考</h1>

<p><a href="http://www.cocoachina.com/b/?p=216">1&#46;Cocoa China 苹果开发中文站</a></p>

<p><a href="">2&#46;《Object-C基础教程》</a></p>
