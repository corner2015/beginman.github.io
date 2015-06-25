---
layout: post
title: "How do imports work in Tornado templates"
description: "How do imports work in Tornado templates"
category: "tornado"
tags: []
---
{% include JB/setup %}
<ul>
    <li>作者：<a href="http://weibo.com/beginman" target="blank">BeginMan</a></li>
    <li>本文地址：http://beginman.github.io</li>
    <li>转载请注明出处</li>
</ul>
<p><a href="http://tornado.readthedocs.org/en/latest/template.html">Tornado Template document:</a></p>

<p><strong>Question Follow:</strong></p>

<blockquote>
  <p>The documentation for Tornado's templating system is poor. Is there a way to import modules? The following does not work:</p>
</blockquote>

<pre><code>{% import os %}
{{ os }}
</code></pre>

<p>This results in <strong>"NameError: global name 'os' is not defined"</strong></p>

<p><strong>Solutions：</strong></p>

<blockquote>
  <p>If I have template B extending template A, putting the import at the top of A works, but putting it at the top of B seems to do nothing.</p>
</blockquote>

<p>This actually makes a kind of sense, now that I'm thinking about it. Since only the stuff in <code>{% block %}</code> tags in B actually gets rendered, it makes sense that it wouldn't even see the <code>{% import %}</code> tags. Maybe putting them inside one of the blocks would do the trick, I'm not sure.</p>

<p>Address:<a href="http://www.quora.com/How-do-imports-work-in-Tornado-templates">http://www.quora.com/How-do-imports-work-in-Tornado-templates</a></p>
