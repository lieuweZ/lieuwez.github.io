---
layout: default
title: Projects
---

<div class="projects-grid">
  {% assign sorted_projects = site.projects | sort: 'order' %}
  {% for project in sorted_projects %}
    <div class="project-card{% if project.image_contain %} project-card--contain{% endif %}">
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
