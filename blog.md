## Bloggin'

{% for category in site.categories %}
  <h3>{{ category[0] }}</h3>
  <ul>
    {% for post in category[1] reversed %}
      {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endunless %}
    {% endfor %}
  </ul>
{% endfor %}
