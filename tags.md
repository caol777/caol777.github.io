---
layout: page
title: Tags
permalink: /tags/
---

# All Tags

<style>
.tags-container { margin: 18px 0; }
.tag-box {
  display: inline-block;
  background: var(--terminal-accent);
  color: var(--terminal-bg) !important;
  padding: 5px 12px;
  border-radius: 4px;
  margin: 5px;
  text-decoration: none !important;
  font-weight: bold;
  text-shadow: none;
}
.tag-box:hover {
  background: var(--terminal-yellow);
  color: var(--terminal-bg) !important;
  text-decoration: none !important;
}
.tag-list-section h2 { scroll-margin-top: 20px; }
</style>

<div class="tags-container">
  {% assign tags = site.tags | sort %}
  {% for tag in tags %}
    <a href="#{{ tag[0] | slugify }}" class="tag-box">#{{ tag[0] }} ({{ tag[1].size }})</a>
  {% endfor %}
</div>

<hr>

<div class="tag-list-section">
{% for tag in tags %}
  <h2 id="{{ tag[0] | slugify }}">#{{ tag[0] }}</h2>
  <ul>
    {% for post in tag[1] %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <small>{{ post.date | date: "%B %-d, %Y" }}</small>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
</div>
