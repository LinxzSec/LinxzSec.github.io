## OSCP

{% for post in site.posts %}
  {% if post.categories contains 'OSCP Journey' %}
    {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endif %}
{% endfor %}
