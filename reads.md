---
layout: default
title: "reads"
---

<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>

<div class="reads-container">
  {% assign book_covers = site.static_files | where_exp: "file", "file.path contains 'assets/reads/2024'" %}

  <h1>2024</h1>

  <div class="book-grid">
    {% for book in book_covers %}
      <div class="book-cover">
        <img src="{{ site.baseurl }}{{ book.path }}" alt="Book cover">
      </div>
    {% endfor %}
  </div>
</div>

<script>
  var elem = document.querySelector('.book-grid');
  var msnry = new Masonry( elem, {
    itemSelector: '.book-cover',
    columnWidth: '.book-cover',
    percentPosition: true
  });
</script>
