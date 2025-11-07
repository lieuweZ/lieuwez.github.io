---
layout: default
title: Projects
---

<div class="projects-grid">
  {% for project in site.projects %}
    <div class="project-card">
      <a href="{{ project.url | relative_url }}">
        {% if project.image %}
          <img src="{{ project.image | relative_url }}" alt="{{ project.title }}">
        {% endif %}
        <h3>{{ project.title }}</h3>
        <p class="project-category">{{ project.category }}</p>
      </a>
    </div>
  {% endfor %}
</div>
