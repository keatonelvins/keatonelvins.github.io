---
layout: default
title: "beaks and claws"
---

<div class="title">
    <h1>{{ page.title }}</h1>
</div>

<section id="posts">
  {% for post in site.posts %}
    <div class="post-widget">
      <a href="{{ post.url }}">
        <img src="{{ site.baseurl }}{{ post.image }}" alt="{{ post.title }}">
        <h2>{{ post.title }}</h2>
        <p>{{ post.category }}</p>
      </a>
    </div>
  {% endfor %}
</section>