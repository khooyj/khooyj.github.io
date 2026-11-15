---
layout: default
title: Articles
description: Writing on AI, security, cloud and engineering leadership.
permalink: /articles/
---

<p class="crumbs"><a href="{{ '/' | relative_url }}">Home</a> › Articles</p>

<h1 class="page-title">Articles</h1>
<p class="lead">Writing on AI, security, cloud and engineering leadership. <a href="{{ '/feed.xml' | relative_url }}">RSS feed</a></p>

<ul class="post-list">
  {% for post in site.posts %}
  <li>
    <a class="post-card" href="{{ post.url | relative_url }}">
      <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%-d %b %Y" }}</time>
      <h2>{{ post.title }}</h2>
      <p>{{ post.description }}</p>
      {% if post.tags.size > 0 %}<ul class="post-tags">{% for tag in post.tags %}<li>{{ tag }}</li>{% endfor %}</ul>{% endif %}
    </a>
  </li>
  {% endfor %}
</ul>
