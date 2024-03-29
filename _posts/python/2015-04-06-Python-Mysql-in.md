---
layout: post
title: "Python Mysql in 操作处理"
description: "Python Mysql in 操作处理"
category: "python"
tags: [python技巧]
---
{% include JB/setup %}

<p>Python Mysql <code>in</code>操作容易引发如下错误：</p>

<pre><code>args=['A','B','C','D']
cursor.execute("select * from tableA where mm in (%s)" % ‘,’.join(args))
</code></pre>

<p>这样会触发Mysql异常：OperationalError: (1292, "Truncated incorrect DOUBLE value: 'A,B,C,D'")</p>

<p>我们需要手动构建的查询参数</p>

<p>解决方案1：</p>

<pre><code>param_list = '%s,' * (len(args) - 1) + '%s'
sql = "select * from tableA where mm in (%s)" % param_list
cursor.execute(sql, args)

#torndb: conn is tornado cursor
conn.execute(sql, *args)
</code></pre>

<p>解决方案2：for python3:</p>

<pre><code>sql='SELECT * FROM tableA WHERE mm IN (%s)' 
in_p=', '.join(list(map(lambda x: '%s', args)))
sql = sql % in_p        # SELECT * FROM tableA WHERE mm IN (%s, %s)

cursor.execute(sql, args)
</code></pre>

<p>解决方案3：for python2:</p>

<pre><code>sql='SELECT * FROM tableA WHERE mm IN (%s)' 
in_p=', '.join(map(lambda x: '%s', args))
sql = sql % in_p        # SELECT * FROM tableA WHERE mm IN (%s, %s)

cursor.execute(sql, args)
</code></pre>

<p>从上述解决方案可知，关键是手动构造: xxx IN (%s, %s), 对于上面的in_p 还有如下方法构造：</p>

<pre><code>import itertools
in_p = ', '.join(itertools.repeat('%s', len(args)))   #%s, %s
</code></pre>
