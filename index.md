---
layout: page
title: 追架构之路，谱全栈人生.
tagline: 
---
{% include JB/setup %}


<div class="posts-list">
    <h3>文章列表</h3>
    <ul class="posts">
        {% for post in paginator.posts %}
            <li><span>{{ post.date | date: '%Y-%m-%d' }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
    </ul>
</div>

<!-- 导航分页链接 -->

<div class="pagination">
  {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path }}" class="previous">Previous</a>
  {% else %}
    <span class="previous">Previous</span>
  {% endif %}
  <span class="page_number ">Page: {{ paginator.page }} of {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path }}" class="next">Next</a>
  {% else %}
    <span class="next ">Next</span>
  {% endif %}
</div>