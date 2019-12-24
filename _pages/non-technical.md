---
permalink: /non-technical
---

## Non-Technical

<ul>
  {% for post in site.categories.Non-technical %}
    {% unless post.draft %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endunless %}
  {% endfor %}
</ul>
