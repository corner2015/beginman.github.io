---
layout: post
title: "CentOS服务器开发初步"
description: "CentOS服务器开发初步"
category: "运维"
tags: [运维]
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p>最讨厌的事情来了，那就是做数据迁移和服务器重新搭建，而且还是两台。我不是专业的运维人员，我特别想知道他们是怎么管理这些服务器集群的，或者他们是怎么进行成百上千台服务器搭建的。</p>

<p>没有办法，只能继续埋头搭建了，这里我总结了我需要做的事情：</p>

<h1>一.安全性配置</h1>

<ul>
<li>更改root密码使之更加健壮</li>
<li>生成普通用户</li>
<li>ssh配置，禁用root登陆，以及更加安全性的配置</li>
<li>运行环境配置如<code>locale</code></li>
<li>搭建防火墙，安装Fail2Ban等防暴力破解工具</li>
<li>做一些端口对外限制</li>
<li>挂载硬盘，分配空间</li>
</ul>

<h1>二.开发环境搭建</h1>

<ul>
<li>操作系统升级软件</li>
<li>升级Python</li>
<li>安装redis，并做好安全性，读写分离，备份等基础配置</li>
<li>安装mongodb,并做好安全性，读写分离，备份等基础配置</li>
<li>安装mysql,并做好安全性，读写分离，备份等基础配置</li>
<li>安装Python第三方库等开发需要的东西</li>
<li>安装其他工具，如beanstalk,rabbitmq等</li>
<li>安装node环境</li>
</ul>

<h1>三.测试环境搭建</h1>

<ul>
<li>搭建SVN服务器</li>
<li>搭建Git服务器</li>
</ul>

<h1>四.部署环境</h1>

<ul>
<li>安装supervisor，并配置好启动文件</li>
<li>安装nginx,罗列出我们需要的模块(如mongo ,etc)进行统一安装，修改配置文件做好反向代理，负载均衡等基础设置.启动该服务保障应用能够正常</li>
</ul>

<p>当然了这都是基本的，我要是想到了其他的就更新上去，这段时间我会总结下各个阶段的搭建方法，作为自己运维的小小成果吧。:-D 囧</p>
