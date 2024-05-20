---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
# Featured Articles

{% for article in site.articles %}
  <h2>
    <a href="{{ article.url }}">
      {{ article.title }}
    </a>
  </h2>
{% endfor %}

# Other Things

{% for article in random.articles %}
  <h2>
    <a href="{{ article.url }}">
      {{ article.title }}
    </a>
  </h2>
{% endfor %}
