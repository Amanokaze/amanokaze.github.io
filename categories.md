---
layout: default
---

{% for category in site.categories %}
    <h1>{{ category | size }}</h1>
    {% for post in site.categories[category] %}
    <ul>
        <li>
            <h3>
            <a href="{{ site.baseurl }}{{ post.url }}">
                {{ post.title }}
            </a>
            <small>{{ post.date | date_to_string }}</small>
            </h3>
        </li>
    </ul>
    {% endfor %}
{% endfor %}