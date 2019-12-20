## Web App

<ul>
  {% for post in site.categories.Web App %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
