---
layout: page
title: Blog
permalink: /blog/
---

<section class="blog-page" aria-labelledby="blog-title">
  <h1 id="blog-title">Blog</h1>
  <p class="blog-intro">Articles and notes on data, AI, and building reliable systems.</p>

  <div class="blog-filters">
    <div class="blog-filter">
      <label for="blog-search">Search posts</label>
      <div class="blog-search-field">
        <svg aria-hidden="true" focusable="false" viewBox="0 0 24 24">
          <circle cx="10.8" cy="10.8" r="6.8"></circle>
          <path d="m16 16 5 5"></path>
        </svg>
        <input id="blog-search" type="search" placeholder="Search posts..." autocomplete="off">
      </div>
    </div>
    <div class="blog-filter">
      {% assign categories = site.categories | sort %}
      {% if categories.size > 7 %}
        <label for="blog-category">Filter by category</label>
        <select id="blog-category">
          <option value="">All categories</option>
          {% for category in categories %}
            <option value="{{ category[0] | downcase | escape }}">{{ category[0] | escape }}</option>
          {% endfor %}
        </select>
      {% else %}
        <span class="blog-filter-label" id="blog-category-label">Filter by category</span>
        <div class="blog-category-filters" role="group" aria-labelledby="blog-category-label">
          <button type="button" class="blog-category-filter" data-category-filter="" aria-pressed="true">All</button>
          {% for category in categories %}
            <button type="button" class="blog-category-filter" data-category-filter="{{ category[0] | downcase | escape }}" aria-pressed="false">{{ category[0] | escape }}</button>
          {% endfor %}
        </div>
      {% endif %}
    </div>
  </div>

  <p id="blog-results" class="blog-results" role="status" aria-live="polite"></p>

  <div id="blog-posts" class="blog-post-list">
    {% for post in site.posts %}
      {% assign post_categories = post.categories | join: "|" %}
      {% assign search_text = post.title | append: " " | append: post.description | append: " " | append: post_categories | downcase %}
      {% assign description = post.description | default: post.excerpt | strip_html | strip_newlines | truncate: 180 %}
      {% assign word_count = post.content | strip_html | number_of_words %}
      {% assign reading_minutes = word_count | plus: 199 | divided_by: 200 | at_least: 1 %}
      <article class="blog-card" data-search="{{ search_text | escape }}" data-categories="{{ post_categories | downcase | escape }}">
        <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
        <div class="blog-card-meta">
          <p class="blog-post-meta">
            <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %-d, %Y" }}</time>
            <span aria-hidden="true">&middot;</span>
            <span>{{ reading_minutes }} min read</span>
          </p>
          {% if post.categories.size > 0 %}
            <ul class="blog-categories" aria-label="Categories">
              {% for category in post.categories %}
                <li>{{ category }}</li>
              {% endfor %}
            </ul>
          {% endif %}
        </div>
        {% if description != "" %}
          <p class="blog-card-description">{{ description }}</p>
        {% endif %}
      </article>
    {% endfor %}
  </div>

  {% if site.posts.size == 0 %}
    <p>No posts have been published yet.</p>
  {% else %}
    <p id="blog-no-results" class="blog-no-results" hidden>No posts match your search and category.</p>
  {% endif %}
</section>

<script>
  (() => {
    const searchInput = document.querySelector("#blog-search");
    const categorySelect = document.querySelector("#blog-category");
    const categoryButtons = Array.from(document.querySelectorAll("[data-category-filter]"));
    const posts = Array.from(document.querySelectorAll(".blog-card"));
    const noResults = document.querySelector("#blog-no-results");
    const resultsStatus = document.querySelector("#blog-results");
    const hasCategoryControl = categorySelect || categoryButtons.length > 0;

    if (!searchInput || !hasCategoryControl || !noResults || !resultsStatus) {
      return;
    }

    const updatePosts = () => {
      const query = searchInput.value.trim().toLowerCase();
      const selectedButton = categoryButtons.find((button) => button.getAttribute("aria-pressed") === "true");
      let category = "";
      if (categorySelect) {
        category = categorySelect.value;
      } else if (selectedButton) {
        category = selectedButton.dataset.categoryFilter;
      }
      let visibleCount = 0;

      posts.forEach((post) => {
        const matchesQuery = (post.dataset.search || "").includes(query);
        const categories = (post.dataset.categories || "").split("|");
        const matchesCategory = !category || categories.includes(category);
        const isVisible = matchesQuery && matchesCategory;

        post.hidden = !isVisible;
        if (isVisible) {
          visibleCount += 1;
        }
      });

      noResults.hidden = visibleCount > 0;
      resultsStatus.textContent = visibleCount === 1 ? "1 post found." : `${visibleCount} posts found.`;
    };

    searchInput.addEventListener("input", updatePosts);
    if (categorySelect) {
      categorySelect.addEventListener("change", updatePosts);
    }

    categoryButtons.forEach((button) => {
      button.addEventListener("click", () => {
        categoryButtons.forEach((candidate) => {
          candidate.setAttribute("aria-pressed", String(candidate === button));
        });
        updatePosts();
      });
    });
  })();
</script>
