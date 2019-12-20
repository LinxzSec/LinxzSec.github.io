## Vulnerablities

<ul>
  {% for post in site.categories.Vulnerabilities %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
