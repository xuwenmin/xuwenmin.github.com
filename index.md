---
layout: page
title: 首页
header : Post Archive
group: navigation
---
{% include JB/setup %}


{% assign posts_collate = site.posts %}
{% include JB/posts_collate %}



### 标签云

<ul class="tag_box inline">
  {% assign tags_list = site.tags %}  
  {% include JB/tags_list %}
</ul>



