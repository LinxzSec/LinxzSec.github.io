## Cryptography Ground Up

<ul>
  {% for post in site.categories.Cryptography-Ground-Up %}
    {% for post in category[1] reversed %}
      {% unless post.draft %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endunless %}
    {% endfor %}
  {% endfor %}
</ul>
