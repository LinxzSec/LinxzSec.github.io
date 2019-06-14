## OSCP

{% for post in site.posts %}
  {% if post.categories contains 'OSCP Journey' %}
    {% unless post.draft %}
      <ul>
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      </ul>
    {% endunless %}
  {% endif %}
{% endfor %}
