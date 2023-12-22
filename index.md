---
layout: home
title: "beaks and claws"
---
<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>

<div class="title">
    <h1>{{ page.title }}</h1>
</div>

{% assign posts_with_images = site.posts | where_exp: "post", "post.image" %}
{% assign total_images = posts_with_images | size %}

{% assign counter = 0 %}
<section id="posts">
  {% for post in site.posts %}
    <div class="post-widget">
      <a href="{{ post.url }}">
        {% if post.image %}
          {% assign idx = total_images | minus: counter %}
          <img src="{{ site.baseurl }}/assets/posts/{{ idx }}.jpg" alt="{{ post.title }}">
          {% assign counter = counter | plus: 1 %}
        {% endif %}
        <div class="post-info">
          <p>{{ post.category }}</p>
          <h2>{{ post.title }}</h2>
          <p>{{ post.date | date_to_string }}</p>
        </div>
      </a>
    </div>
  {% endfor %}
</section>

<script>
  var elem = document.querySelector('#posts');
  var msnry = new Masonry( elem, {
    itemSelector: '.post-widget',
    columnWidth: '.post-widget',
    percentPosition: true
  });
</script>