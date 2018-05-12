---
title: All notes
---

## All notes list

All my notes are listed here:

<ul>
  {% for post in site.posts %}
{% if post.title %}
{% capture linktext %}{{ post.title }}{% endcapture %}
{% else %}
{% capture linktext %}Note{% endcapture %}
{% endif %}
    <li>
      <a href="{{ post.url }}">
        {{ linktext }}
      </a>
      - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
    </li>
  {% endfor %}
</ul>


[Back to the main page](README.md).
