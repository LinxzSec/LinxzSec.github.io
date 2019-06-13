## OSCP

{% for page in site.pages %}
  {% if page.categories contains 'OSCP Journey' %}
    {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endif %}
{% endfor %}
