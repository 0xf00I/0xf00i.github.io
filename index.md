---
layout: main
permalink: /
---

# Home 
A security blog focusing on malware development, reverse engineering, and binary exploitation.

---

## Recent Posts

<ul class="related-posts">

{% assign blog_posts = site.posts | where: 'blog_post', true %}
{% for post in blog_posts limit:2 %}
    <li class="main-page-list">
        <h4>
            <div style="display: inline-block; width: 90px">
                <small>{{ post.date | date: "%Y-%m-%d" }}</small>
            </div>
            <div id="main-page-blogs-list">
                <a class="una" href="{{ site.baseurl }}{{ post.url }}">
                    <span>{{ post.title }}</span>
                </a>
            </div>
        </h4>
    </li>
    {% if forloop.last %}</ul>{% endif %}
{% endfor %}
