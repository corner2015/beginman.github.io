---
layout: post
title: "记一次处理服务器异常之旅"
description: "记一次处理服务器异常之旅"
category: "life"
tags: [人生]
---
{% include JB/setup %}

<blockquote>
  <p>呵呵，搞服务器开发挺有压力的，稍微不注意的话服务器就可能宕机，如果牵涉公司主营业务的话，就感觉自己经历一场生死。对于我而已，即做后台开发，又做服务器运维，还兼任DBA,处理web前端。在小公司里身兼众职，难免会碰到很多问题，或者不注意就发生的故障。摆好心态，制定好应急方案，架设好自动化工具和安全监测是很有必要的。</p>
</blockquote>

<h2>1.问题回放</h2>

<p>今天运营反馈一个问题，用户身份认证失败，查看服务器发现linux磁盘爆满，查找大文件，发现罪魁祸首的是一个redis_6379.log的日志，居然占了64G. 如果是原来的话，我会直接删除它，就ok了，但是本着负责任的态度，肯定要找出源头啊，于是就立马修改了redis配置文件，然后重启redis。接着连带的服务器上的Python服务和C#服务都不能正常运行了，输入redis命令也没有反应。到这里我才明白，<strong>redis加载配置文件失败，如果更改配置文件要先停掉redis服务，然后再重启redis服务</strong></p>

<p>通过Charles抓包工具发现<strong>C#和python服务都大量的阻塞着，此刻解决办法是重启相关服务</strong>, 重启后发现还是有问题，一个请求还是阻塞着，心想肯定是redis的问题。</p>

<p>进入redis命令行，输入命令发现redis不再响应命令，输入<code>redis-cli shutdown</code> 也无济于事，kill 掉redis进程后再重启也不行，郁闷。于是乎就重启了机器，重启的后果是其他服务器上的服务没法运行了。这里我做的排查如下：</p>

<ol>
<li>redis是否正常启动，更改的配置文件是否生效</li>
<li>Charles抓包发现请求还是阻塞着</li>
<li>重启Python服务，Python服务运行正常</li>
<li>重启C#服务，C#服务不再阻塞但请求失败</li>
<li>查看C#服务的日志，发现原来beanstalkd没有启动，赶紧启动，再重启C#服务，请求成功</li>
</ol>

<p>妈蛋，心想着老板拿着刀架在脖子上威胁的场景，我连中午饭都没吃好。</p>

<p><img src="http://beginman.qiniudn.com/Downtime.jpeg" alt="" /></p>

<!--more-->

<h2>2.吃一堑，长一智</h2>

<p>由于公司交接文档少之又少，对8台服务器到底运行了哪些服务，哪些程序，哪些部署并不是多清楚，于是乎就导致了问题的出现。这里反省如下：</p>

<ol>
<li>遇到问题不要心急，静下心来想一想问题的所在</li>
<li>一定要先把各个服务器运行的东西弄清楚，部署的服务整理好，比如我就不知道那台宕掉的服务器还运行着beanstald服务。</li>
<li>要先从修复风险最小的方案入手，如我可以直接删掉那个redis_6379.log 罪魁祸首的文件，或者写个脚本定时删除。而不是选择自己陌生的解决方案，如直接修改redis配置文件，不友好的重启方式</li>
<li>在确定要重启服务器之前，先把运行的服务，如Python服务，<code>ps -ef | grep python</code>, redis服务，mysql服务，beanstalkd服务，需要开机启动的服务等记录下来，然后再进行重启，重启之后立马运行之前记录下的服务</li>
<li>千万别忽略log日志，写的服务一定要加上log日志，这样有利于异常排除</li>
<li>自动化运维的平台还是很需要的，时常做灾难预警，自动修复</li>
</ol>
