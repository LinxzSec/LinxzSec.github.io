## Non-Technical

<ul>
  {% for post in site.categories.Non-Technical %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
