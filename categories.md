---
layout: default
title: Categories
---

<div id="articles">
{% assign pages_list = site.pages %}
{% for node in pages_list %}
    {% if node.title != null %}
    {% if node.layout == "category" %}
        <h1><a class="category-link {% if page.url == node.url %} active{% endif %}"
        href="{{ site.baseurl }}{{ node.url }}">{{ node.title }}</a></h1>

        <ul class="posts-list">
        
        {% for post in site.categories[node] %}
            <li>
            <h3>
                <a href="{{ site.baseurl }}{{ post.url }}">
                {{ post.title }}
                </a>
                <small>{{ post.date | date_to_string }}</small>
            </h3>
            </li>
        {% endfor %}
        
        </ul>

    {% endif %}
    {% endif %}
{% endfor %}
</div>