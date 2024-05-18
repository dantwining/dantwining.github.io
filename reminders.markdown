---
layout: home
permalink: /reminders/
---

{% for article in site.random %}
  <h2>
    <a href="{{ article.url }}">
      {{ article.title }}
    </a>
  </h2>
{% endfor %}

