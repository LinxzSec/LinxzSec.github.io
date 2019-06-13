## OSCP

{% for post in site.post %}
  {% if post.categories contains 'OSCP Journey' %}
    {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
{% endfor %}
