## Bloggin'

{% unless page.content == '' %}
  <p>{{ page.content }}</p>
{% endunless %}

{% for post in site.categories.[page.category] %}
  <h2><a href=""></a></h2>
  <p></p>
{% endfor %}
