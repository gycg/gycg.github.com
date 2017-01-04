---
layout: page
title: "Tags"
description: "Labels of Blog"  
header-img: "img/facebook.jpg"  
---

<!--

<div id='tag_cloud'>
{% for tag in site.tags %}
<a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}</a>
{% endfor %}
</div>

*** 
✈

<ul class="listing">
{% for tag in site.tags %}
  <li class="listing-seperator" id="{{ tag[0] }}">{{ tag[0] }}</li>
{% for post in tag[1] %}
  <li class="listing-item">
  <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
  <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
{% endfor %}
</ul>

<script src="/media/js/jquery.tagcloud.js" type="text/javascript" charset="utf-8"></script> 
<script language="javascript">
$.fn.tagcloud.defaults = {
    size: {start: 1, end: 1, unit: 'em'},
      color: {start: '#f8e0e6', end: '#ff3333'}
};

$(function () {
    $('#tag_cloud a').tagcloud();
});
</script>
-->
<div id="post-list">
{% for tag in site.tags %}
<h2 id="{{ tag[0] }}">{{ tag[0] | capitalize }}</h2>
<ul class="post-list">
{% assign pages_list = tag[1] %}
{% for post in pages_list %}
{% if post.title != null %}
{% if group == null or group == post.group %}
<li><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}<span class="entry-date"> ~ <time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %d, %Y" }}</time></a></li>
{% endif %}
{% endif %}
{% endfor %}
{% assign pages_list = nil %}
{% assign group = nil %}
</ul>
{% endfor %}
</div>
