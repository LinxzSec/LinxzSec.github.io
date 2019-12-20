## Cryptography Ground Up

<ul>
  {% for post in site.categories.Crypto Ground Up %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
