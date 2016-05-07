---
title: News
layout: default
permalink: /news/
---

<ul>
  {% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{post.date | date: '%B %d, %Y'}} {{ post.title }}</a><br/>
    {{ post.content }}
    <br/>
    <br/>
  </li>
  {% endfor %}
</ul>

