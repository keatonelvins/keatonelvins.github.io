---
layout: default
title: "home"
---
<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>
<script src="https://unpkg.com/imagesloaded@5/imagesloaded.pkgd.min.js"></script>

<div class="landing">
    <img src="{{ site.baseurl }}/assets/art/cover.jpg" alt="cover art">
</div>

{% assign posts_with_images = site.posts | where_exp: "post", "post.image" %}

<section id="posts">
  {% for post in site.posts %}
    <div class="post-widget">
      <a href="{{ post.url }}">
        {% if post.image %}
          <img src="{{ site.baseurl }}/assets/thumbnails/{{ post.image_name }}.jpg" alt="{{ post.title }}">
        {% endif %}
        <div class="post-info">
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

  var imgLoad = imagesLoaded(elem);
  imgLoad.on('done', function() {
    msnry.layout();

    document.querySelectorAll('.post-widget').forEach((post, index) => {
      setTimeout(() => {
        post.style.opacity = '1';
      }, index * 50);
    });
  });
</script>
