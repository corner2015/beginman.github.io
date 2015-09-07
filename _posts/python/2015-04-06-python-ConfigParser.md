---
layout: post
title: "python ConfigParser的应用与分析"
description: "python ConfigParser的应用与分析"
category: "Python"
tags: [Python]
---
{% include JB/setup %}

<h2>1.常见的配置文件</h2>

<p><code>ConfigParser</code>模块用于设置和解析<code>.ini</code>文件，.ini 文件是Initialization File的缩写，即初始化文件，有时候，INI文件也会以不同的扩展名，如“.CFG”、“.CONF”、或是“.TXT”代替。</p>

<p>它的格式如下：</p>

<pre><code>; last modified 1 April 分号后表示注释信息  
[owner] ;表示节([section])
name=John Doe  ;参数（k-v形式： name=Jonn, 或者 name: jone）
organization=Acme Products

[database]
server=192.0.2.42 ; use IP address in case network  name resolution is not working
port=143
file = "acme payroll.dat"
</code></pre>

<!--more-->

<p>对于<code>cfg</code>格式文件，大多数情况下，很多程序都要保存用户的设置，办法有很多：注册表，日志文件·..... 而很多程序都使用了一个专用的文件。为了方便起见，常常命名为*.cfg，有时甚至直接命名为Config.cfg。</p>

<h2>2.ConfigParser类解析ini,cfg等配置文件</h2>

<p>该模块下有三个类：</p>

<pre><code>ConfigParser.RawConfigParser([defaults[, dict_type[, allow_no_value]]])
</code></pre>

<p>这是基本的配置类，该类不支持魔术插入。方法如下：</p>

<p>RawConfigParser.defaults() 返回一个包含全部实例的字典</p>

<p>RawConfigParser.sections() 返回一个包含有效section的列表，DEFAULT不包含在该列表中</p>

<p>RawConfigParser.add_section(section) 增加一个section，如果section存在DuplicateSectionError会被触发。</p>

<p>RawConfigParser.has_section(section) 判断section是否在配置文件中存在</p>

<p>RawConfigParser.options(section) 返回section中可用的options 列表</p>

<p>RawConfigParser.has_option(section, option) 判断section中是否存在options</p>

<p>RawConfigParser.read(filenames) 读入被解析的配置文件</p>

<p>RawConfigParser.readfp(fp[, filename]) 读入并解析配置文件</p>

<p>RawConfigParser.get(section, option) 获取section中option的值</p>

<p>RawConfigParser.getint(section, option) 已整形返回option的值</p>

<p>RawConfigParser.getfloat(section, option) 同理上面，返回float</p>

<p>RawConfigParser.getboolean(section, option)</p>

<p>RawConfigParser.items(section) 以列表(name,value)的形式返回section中的每个值</p>

<p>RawConfigParser.set(section, option, value) 如果section存在，则设置该option和value，否则引起NoSectionError.</p>

<p>RawConfigParser.write(fileobject) 配置写入配置文件中
RawConfigParser.remove_option(section, option) 移除section中的option，如果section不存在，引起NoSectionError，移除后返回True，否则返回False</p>

<p>RawConfigParser.remove_section(section) 移除section，返回True/False
RawConfigParser.optionxform(option)</p>

<pre><code>ConfigParser.ConfigParser([defaults[, dict_type[, allow_no_value]]])
</code></pre>

<p>该类是RawConfigParser的派生类。支持魔术插入，增加了get和items方法。</p>

<p>ConfigParser.get(section, option[, raw[, vars]]) 获取section中option的值，</p>

<p>ConfigParser.items(section[, raw[, vars]]) 返回一个由(name,value)组成的列表对</p>

<pre><code>ConfigParser.SafeConfigParser([defaults[, dict_type[, allow_no_value]]])
</code></pre>

<p>该类是ConfigParser的派生类，支持更多的魔术插入。</p>

<p>SafeConfigParser.set(section, option, value) 如果section存在，则设置option的值。value必须是string。</p>

<p>生成ini配置文件</p>

<pre><code>#!/usr/bin/python 
import ConfigParser

conf=ConfigParser.RawConfigParser()

conf.add_section('section1')
conf.add_section('section2')

conf.set('section1','name1','guol')
conf.set('section1','name2','alex')
conf.set('section2','name3','polo')
conf.set('section2','name4','mark')

conffile=open('file.ini','wb')
conf.write(conffile)
</code></pre>

<p>结果如下：</p>

<p><img src="http://static.oschina.net/uploads/space/2013/0518/152811_CjrX_123777.jpg" alt="" /></p>

<p>解析ini配置文件</p>

<pre><code>import ConfigParser

conf=ConfigParser.RawConfigParser()


conf.read('file.ini')
if conf.has_section('section2'):
    print 'Exist section2'
if conf.has_option('section2','name3'):
    print 'section2 has opetion name3, is value' + ' ' +conf.get('section2','name3')
</code></pre>

<p><img src="http://static.oschina.net/uploads/space/2013/0518/153037_zAyE_123777.jpg" alt="" /></p>

<p>魔术插入</p>

<pre><code>import ConfigParser

conf1=ConfigParser.ConfigParser()
conf1.read('file.ini')
conf2=ConfigParser.RawConfigParser()
conf2.read('file.ini')
print 'Use ConfigParser()'
print '''conf1.get('section3','name3',0)'''
print conf1.get('section3','name3',0)
print '''conf1.get('section3','name3',1)'''
print conf1.get('section3','name3',1)
print '================================'
print 'Use RawConfigParser()'
print '''conf2.get('section3','name3')'''
print conf2.get('section3','name3')
</code></pre>

<p><img src="http://static.oschina.net/uploads/space/2013/0518/153421_Zi9j_123777.jpg" alt="" /></p>

<h2>3.运用实例</h2>

<p>通常在配置python应用的时候，不妨可以使用它，如下摘自Github</p>

<pre><code>#!/usr/bin/env python
#-*- coding:utf-8 -*-
import ConfigParser
import MySQLdb
import getpass
import hashlib
import re
import subprocess

print('=' * 50)
print('安装脚本'.center(54))
print('=' * 50)

print
#初始化ConfigParse实例
config = ConfigParser.ConfigParser()
user = raw_input('请输入连接数据库的用户名: ')
passwd = getpass.getpass('请输入连接数据库的密码: ')

with MySQLdb.connect(user=user, passwd=passwd) as c:
    print('连接成功!')

    print

    while True:
        database = raw_input('请输入需要创建的数据库名: ')
        mysqlUser = raw_input('请输入该博客连接数据库的用户名: ')
        mysqlPassword = getpass.getpass('请输入该博客连接数据库的密码: ')
        confirm = getpass.getpass('请再输入一次: ')
        if confirm == mysqlPassword:
            break
        else:
            print

    #执行grant高级DBA管理Mysql中所有数据库的权限
    c.execute("grant all privileges on %s.* to '%s'@'localhost' identified by '%s'" %
        (database, mysqlUser, mysqlPassword))
    c.execute('create database %s' % database)

    #config设置
    config.add_section('mysql')  #设置 mysql节点
    config.set('mysql', 'user', mysqlUser)  #在该节点中添加k-v
    config.set('mysql', 'password', mysqlPassword)
    config.set('mysql', 'database', database)

    #创建一个新的进程用于重定向数据道schema.sql
    subprocess.call("mysql --user=%s --password=%s --database=%s &lt; schema.sql" %
        (mysqlUser, mysqlPassword, database), shell=True)

    print('数据库初始化成功!')

    print

    while True:
        while True:
            adminUsername = raw_input('请输入博客管理员的用户名: ')
            if not re.match(r'^[\w\d]+$', adminUsername):
                print('用户名只能由字母和数字组成')
                print
            else:
                break

        adminPassword = getpass.getpass('请输入博客管理员的密码: ')
        confirm = getpass.getpass('请再输入一次: ')
        if confirm == adminPassword:
            break
        else:
            print
    adminPassword = hashlib.md5(adminPassword).hexdigest()
    c.execute('use %s' % database)
    c.execute("insert into auth (username, password) values ('%s', '%s')" %
        (adminUsername, adminPassword))

    print('管理员用户创建完成!')

    print

    hostname = raw_input('请输入博客域名: ')
    port = raw_input('请输入程序监听的端口号: ')
    user = raw_input('请输入博主的名字（默认: Adonis Ling）: ')
    home_title = raw_input("请输入主页的标题（默认: Adonis's Blog ）: ")
    photo = raw_input('请输入博主头像的文件名(默认: talent.jpg）')

    if len(user) == 0:
        user = 'Adonis Ling'
    if len(home_title) == 0:
        home_title = "Adonis's Blog"
    if len(photo) == 0:
        photo = 'talent.jpg'

    #添加blog节点信息
    config.add_section('blog')
    config.set('blog', 'hostname', hostname)
    config.set('blog', 'port', port)
    config.set('blog', 'user', user)
    config.set('blog', 'home_title', home_title)
    config.set('blog', 'photo', photo)

    # 将配置信息写入blog.cfg文件中
    with open('blog.cfg', 'w') as cfg:
        config.write(cfg)

    print('配置文件生成成功!')

    print

    raw_input('Press any key to continue...')
</code></pre>

<p><strong>参考：<a href="http://my.oschina.net/guol/blog/131816">Python2.7 ConfigParser</a></strong></p>
