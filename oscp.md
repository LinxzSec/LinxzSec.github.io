## OSCP

{% for post in site.posts %}
  {% if post.categories contains 'OSCP Journey' %}
    <ul>
    {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
    </ul>
  {% endif %}
{% endfor %}
