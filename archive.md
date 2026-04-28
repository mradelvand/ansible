---
layout: page
title: Blog Archive
---

{% for category in site.categories %}
  <h2 id="{{ category[0] | slugify }}">{{ category[0] }}</h2>
  <ul>
    {% for post in category[1] %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
