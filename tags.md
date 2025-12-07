---
layout: page
title: Tags
permalink: /tags/
---

# All Tags

<style>
.tag-box {
  display: inline-block;
  background: var(--terminal-text);
  color: #000000;
  padding: 5px 10px;
  border-radius: 4px;
  margin: 5px;
  text-decoration: none;
  font-weight: bold;
}
.tag-box:hover {
    background: var(--terminal-yellow);
    color: var(--terminal-bg) !important;
}
</style>

<div class="tags-container">
  {% assign tags = site.tags | sort %}
  {% for tag in tags %}
    <a href="#{{ tag[0] | slugify }}" class="tag-box">#{{ tag[0] }} ({{ tag[1].size }})</a>
  {% endfor %}
</div>

<hr>

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
