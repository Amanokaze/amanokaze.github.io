---
layout: default
title: Categories
---

<div id="articles">
{% assign pages_list = site.pages %}
{% for node in pages_list %}
    {% if node.title != null %}
    {% if node.layout == "category" %}
        <h3><a class="category-link {% if page.url == node.url %} active{% endif %}"
        href="{{ site.baseurl }}{{ node.url }}">{{ node.title }}</a></h3>

        <ul class="posts-list">
        {% assign category = node.category | default: node.title %}
        
        {% for post in site.categories[category] %}
            <li>
            <h3>
                <a href="{{ site.baseurl }}{{ post.url }}">
                {{ post.title }}
                </a>
                <span class="date">{{ post.date | date: '%B %d, %Y' }}
                    • <a href="https://amanokaze.github.io{{ post.url }}#disqus_thread">0 Comments</a>
                </span>
            </h3>
            </li>
        {% endfor %}
        
        </ul>

    {% endif %}
    {% endif %}
{% endfor %}
</div>