---
permalink: /ctf
---

## CTF

<ul>
  {% for post in site.categories.CTF %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
