---
layout: home
title: "keaton elvins"
---
<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>

{% assign posts_with_images = site.posts | where_exp: "post", "post.image" %}
{% assign total_images = posts_with_images | size %}

{% assign counter = 0 %}
<section id="posts">
  {% for post in site.posts %}
    <div class="post-widget">
      <a href="{{ post.url }}">
        {% if post.image %}
          {% assign idx = total_images | minus: counter %}
          <img src="{{ site.baseurl }}/assets/thumbnails/{{ idx }}.jpg" alt="{{ post.title }}">
          {% assign counter = counter | plus: 1 %}
        {% endif %}
        <div class="post-info">
          <p>{{ post.category }}</p>
          <h2>{{ post.title }}</h2>
          <p>{{ post.date | date_to_string | downcase}}</p>
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