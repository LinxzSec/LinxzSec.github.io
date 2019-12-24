---
layout: default
title: Post by Category
---

<div>
    {% assign categories = site.categories | sort %}
    {% for category in categories %}
        <ul>
            <span class="site-tag">
                <li><a href="{{ category | first | slugify }}">
                        {{ category[0] | replace:'-', ' ' }} ({{ category | last | size }})
                </a></li>
            </span>
        </ul>
    {% endfor %}
</div>
