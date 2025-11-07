---
layout: default
title: Contact
permalink: /contact/
---

<div class="page-content contact-page">
    <h2>Contact</h2>
    
    <p>I'd love to hear from you! Feel free to reach out through any of the following channels:</p>
    
    <div class="contact-info">
        <p><strong>Email:</strong> {{ site.email }}</p>
        <p><strong>GitHub:</strong> <a href="https://github.com/{{ site.github_username }}">@{{ site.github_username }}</a></p>
        {% if site.twitter_username %}
        <p><strong>Twitter:</strong> <a href="https://twitter.com/{{ site.twitter_username }}">@{{ site.twitter_username }}</a></p>
        {% endif %}
    </div>
</div>
