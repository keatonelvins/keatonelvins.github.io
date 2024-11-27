---
layout: default
title: index
---

<ul class="posts-list">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        <div class="posts-list-entry">
          <h2>{{ post.title }}</h2>
          <div class="post-tag">
            <h3>{{ post.tag }}</h3>
          </div>
          <p>{{ post.date | date_to_string | downcase }}</p>
        </div>
      </a>
    </li>
  {% endfor %}
</ul>
