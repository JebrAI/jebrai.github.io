---
layout: default
title: Projects
---

# Projects

Projects WIP:

<div class="projects-grid">
  {% for project in site.projects %}
  <a href="{{ project.url | relative_url }}" class="project-card">
    <h3>{{ project.title }}</h3>
    <p>{{ project.description }}</p>
    <div class="project-meta">
      {% if project.tags %}
      <div class="project-tags">
        {% for tag in project.tags %}
        <span class="tag">{{ tag }}</span>
        {% endfor %}
      </div>
      {% endif %}
      <span class="read-more">Read more</span>
    </div>
  </a>
  {% endfor %}
</div>
