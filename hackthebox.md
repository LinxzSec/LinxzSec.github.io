{% for post in site.categories.hackthebox %}
  {% unless post.draft %}
    <li>{{ post.title }}</li>
  {% endunless %}
{% endfor %}
