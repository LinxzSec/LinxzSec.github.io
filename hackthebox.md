{% for post in site.categories.HackTheBox %}
  {% unless post.draft %}
    <li>{{ post.title }}</li>
  {% endunless %}
{% endfor %}
