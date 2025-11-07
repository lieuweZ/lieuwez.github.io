---
layout: default
title: Blog
permalink: /blog/
---

<div class="page-content">
    <h2>Blog</h2>
    
    <div class="blog-posts">
        {% for post in site.posts %}
            <article class="blog-post-preview">
                <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
                <p class="post-date">{{ post.date | date: "%B %d, %Y" }}</p>
                <p>{{ post.excerpt }}</p>
                <a href="{{ post.url | relative_url }}" class="read-more">Read more →</a>
            </article>
        {% endfor %}
    </div>
</div>
