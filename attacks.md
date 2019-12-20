## Attacks

<ul>
  {% for post in site.categories.Attacks %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
