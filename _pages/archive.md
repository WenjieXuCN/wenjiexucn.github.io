---
layout: page
permalink: /archive/
title: Archive
description: A chronological archive of all my blog posts.
nav: false
nav_order: 2
---

<div class="archive-container">
  <div class="archive-legend">
    <a id="updateFilter" href="#">
      <span class="updated-marker">*</span><span class="filter-text">Filter Updated</span>
    </a>
  </div>

  <div class="archive-sidebar">
    <div class="archive-toc">
      <h3>Years</h3>
      <ul id="year-list">
        {%- assign date_format = "%Y" -%}
        {%- assign sorted_posts = site.posts | sort: 'date' | reverse -%}
        {% assign grouped_posts = sorted_posts | group_by_exp: "post", "post.date | date: date_format" %}
        {% for year in grouped_posts %}
          <li><a href="#{{ year.name }}" data-year="{{ year.name }}">{{ year.name }}</a></li>
        {% endfor %}
      </ul>
    </div>

    <div class="archive-tags">
      <h3>Tags</h3>
      <div class="tag-cloud">
        {% for tag in site.tags %}
          <a href="{{ tag[0] | slugify | prepend: '/blog/tag/' | relative_url }}">
            {{ tag[0] }} <span class="tag-count">({{ tag[1].size }})</span>
          </a>
        {% endfor %}
      </div>
    </div>

    <div class="archive-categories">
      <h3>Categories</h3>
      <div class="category-cloud">
        {% for category in site.categories %}
          <a href="{{ category[0] | slugify | prepend: '/blog/category/' | relative_url }}">
            {{ category[0] }} <span class="category-count">({{ category[1].size }})</span>
          </a>
        {% endfor %}
      </div>
    </div>

  </div>

  <div class="archive">
    {% assign sorted_posts = site.posts | sort: 'last_updated', 'last' | sort: 'date' | reverse %}
    {% assign current_year = "" %}
    {% for post in sorted_posts %}
      {% assign post_date = post.last_updated | default: post.date %}
      {% assign display_year = post.date | date: "%Y" %}
      
      {% if display_year != current_year %}
        {% if current_year != "" %}
          </ul>
        {% endif %}
        <h2 id="{{ display_year }}" class="archive-year-header">{{ display_year }}</h2>
        <ul class="post-list">
        {% assign current_year = display_year %}
      {% endif %}
      
      <li class="post-item{% if post.last_updated %} updated{% endif %}" id="{{ post.date | date: '%Y-%m-%d' }}">
        <span class="post-date">{{ post_date | date: "%b %-d" }}</span>
        <a class="post-title" href="{{ post.url | relative_url }}">
          {{ post.title | escape }}
          {% if post.last_updated %}
            <span class="updated-marker" title="Updated on {{ post.last_updated | date: '%Y-%m-%d' }}">*</span>
          {% endif %}
        </a>
      </li>
    {% endfor %}
    {% if current_year != "" %}
      </ul>
    {% endif %}
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const yearList = document.getElementById('year-list');
  const archiveHeaders = document.querySelectorAll('.archive-year-header');
  const updateFilter = document.getElementById('updateFilter');
  const postItems = document.querySelectorAll('.post-item');
  let filterActive = false;
  let isInitialLoad = true;

  const observer = new IntersectionObserver((entries) => {
    if (!isInitialLoad) {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const year = entry.target.id;
          highlightYear(year);
        }
      });
    }
  }, { threshold: 0.5 });

  archiveHeaders.forEach(header => observer.observe(header));

  yearList.addEventListener('click', function(e) {
    if (e.target.tagName === 'A') {
      e.preventDefault();
      const targetId = e.target.getAttribute('href').slice(1);
      const targetElement = document.getElementById(targetId);
      if (targetElement) {
        isInitialLoad = false;
        targetElement.scrollIntoView({ behavior: 'smooth' });
      }
    }
  });

  updateFilter.addEventListener('click', function() {
    filterActive = !filterActive;
    this.classList.toggle('active', filterActive);
    postItems.forEach(item => {
      if (filterActive) {
        item.style.display = item.classList.contains('updated') ? '' : 'none';
      } else {
        item.style.display = '';
      }
    });
    updateFilterText();
  });

  function highlightYear(year) {
    const links = yearList.querySelectorAll('a');
    links.forEach(link => {
      if (link.getAttribute('data-year') === year) {
        link.classList.add('active');
      } else {
        link.classList.remove('active');
      }
    });
  }

  function updateFilterText() {
    const filterText = updateFilter.querySelector('.filter-text');
    filterText.textContent = filterActive ? 'Show all' : 'Filter updated';
  }

  function removeAllHighlights() {
    const links = yearList.querySelectorAll('a');
    links.forEach(link => link.classList.remove('active'));
  }

  // 页面加载时调用
  removeAllHighlights();

  // 监听滚动事件，确保正确高亮当前年份
  window.addEventListener('scroll', function() {
    if (!isInitialLoad) {
      let currentYear = null;
      archiveHeaders.forEach(header => {
        const rect = header.getBoundingClientRect();
        if (rect.top <= window.innerHeight / 2 && rect.bottom >= window.innerHeight / 2) {
          currentYear = header.id;
        }
      });
      if (currentYear) {
        highlightYear(currentYear);
      } else {
        removeAllHighlights();
      }
    }
  });

  // 延迟设置 isInitialLoad 为 false
  setTimeout(() => {
    isInitialLoad = false;
    // 触发一次滚动事件，以正确设置初始高亮状态
    window.dispatchEvent(new Event('scroll'));
  }, 100);

  if ('loading' in HTMLImageElement.prototype) {
    const images = document.querySelectorAll('img[loading="lazy"]');
    images.forEach(img => {
      img.src = img.dataset.src;
    });
  }
});
</script>
