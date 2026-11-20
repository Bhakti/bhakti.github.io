---
layout: page
title: Articles
section: KNOWLEDGE LIBRARY
description: Practical notes, architecture patterns and lessons from building enterprise technology.
permalink: /blog/
---
<div class="blog-list">
{% for post in site.posts %}
  <article class="blog-card">
    <p class="post-category">{{ post.categories | join: " · " }}</p>
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p>{{ post.description | default: post.excerpt | strip_html | truncate: 190 }}</p>
    <div class="blog-card-meta">{{ post.date | date: "%d %b %Y" }} <span>·</span> {{ post.reading_time | default: "8 min read" }} <a href="{{ post.url | relative_url }}">Read article ↗</a></div>
  </article>
{% endfor %}
</div>
