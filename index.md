---
layout: page
title: Steve Kennaird's Development Blog
tagline: Adventures in web development
---
{% include JB/setup %}
  
## Latest Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>