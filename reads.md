---
layout: default
title: reads
---

<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>
<script src="https://unpkg.com/imagesloaded@5/imagesloaded.pkgd.min.js"></script>

<div class="reads-container">
  {% assign covers_2024 = site.static_files | where_exp: "file", "file.path contains 'assets/reads/2024'" %}
  {% assign covers_2025 = site.static_files | where_exp: "file", "file.path contains 'assets/reads/2025'" %}
  {% assign reading = site.static_files | where_exp: "file", "file.path contains 'assets/reads/reading'" %}

  <div class="book-recommendation-form">
    <h2>anonymous book recs!! (go go go)</h2>
    <form id="bookRecommendationForm" action="https://formspree.io/f/xdkollpk" method="POST">
      <div class="form-group">
        <input type="text" id="bookTitle" name="bookTitle" placeholder="don't be shy, you're already here..." required>
      </div>
      <button type="submit">send!</button>
    </form>
  </div>

  <div class="book-grid">
    {% for book in reading %}
      <div class="book-cover">
        <img src="{{ site.baseurl }}{{ book.path }}" alt="Book cover">
        <div class="book-year">currently reading</div>
      </div>
    {% endfor %}
    {% for book in covers_2025 %}
      <div class="book-cover">
        <img src="{{ site.baseurl }}{{ book.path }}" alt="Book cover">
        <div class="book-year">read in 2025</div>
      </div>
    {% endfor %}
    {% for book in covers_2024 %}
      <div class="book-cover">
        <img src="{{ site.baseurl }}{{ book.path }}" alt="Book cover">
        <div class="book-year">read in 2024</div>
      </div>
    {% endfor %}
  </div>
</div>

<script>
  // Initialize Masonry for all book grids
  document.querySelectorAll('.book-grid').forEach(elem => {
    var msnry = new Masonry(elem, {
      itemSelector: '.book-cover',
      columnWidth: '.book-cover',
      percentPosition: true
    });

    // Handle image loading for each grid
    var imgLoad = imagesLoaded(elem);
    imgLoad.on('done', function() {
      msnry.layout();

      elem.querySelectorAll('.book-cover').forEach((cover, index) => {
        setTimeout(() => {
          cover.style.opacity = '1';
        }, index * 50);
      });
    });
  });

  document.getElementById('bookRecommendationForm').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const form = this;
    const formData = new FormData(form);
    form.reset();

    fetch(form.action, {
      method: 'POST',
      body: formData,
      headers: {
        'Accept': 'application/json'
      }
    })
    .then(response => response.json())
    .then(data => {
      if (data.ok) {
        const alert = document.createElement('div');
        alert.className = 'alert';
        alert.textContent = 'thank you! <333';
        document.body.appendChild(alert);

        setTimeout(() => alert.classList.add('show'), 100);

        setTimeout(() => {
          alert.classList.remove('show');
          setTimeout(() => alert.remove(), 300);
        }, 4000);
      }
    })
    .catch(error => {
      console.error('Error:', error);
      alert('oop, something went wrong :^/');
    });
  });
</script>
