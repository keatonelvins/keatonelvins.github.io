---
layout: default
title: Index
---

<ul class="posts-list">
  {% for post in site.posts %}
    <li>
      <div class="posts-list-entry">
        <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
        <p>{{ post.date | date_to_string }}</p>
      </div>
    </li>
  {% endfor %}
</ul>