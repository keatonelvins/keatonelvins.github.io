---
layout: posts
title: index
---

<ul class="posts-list">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        <div class="posts-list-entry">
          <h2>{{ post.title }}</h2>
          <p>{{ post.date | date_to_string | downcase }}</p>
        </div>
      </a>
    </li>
  {% endfor %}
</ul>