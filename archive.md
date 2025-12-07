---
layout: page
title: Blog Archive
---

{% for post in site.posts %}
  <ul>
    <li>
      <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
    </li>
  </ul>
{% endfor %}
