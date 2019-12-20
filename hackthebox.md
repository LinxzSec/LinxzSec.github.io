{% for post in site.categories.HackTheBox %}
  {% unless post.draft %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endunless %}
{% endfor %}
