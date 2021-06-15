---
permalink: /
excerpt: "Time Line"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

<div class="container">
{% assign pos_proj = 0 %}
{% assign pos_post = 0 %}
{% assign size_proj = site.projects.size %}
{% assign size_post = site.posts.size %}
{% assign size = size_proj | plus: size_post %}

{% for num in (1..size) %}
  {% if pos_proj >= size_proj %}
    {% assign post = site.posts[pos_post] %}
    {% assign pos_post =  pos_post | plus: 1 %}
    {% assign date = post.date | split: " " %}
    <div class="timeline-item" date-is='{{ date[0] }}'>
      <h1>{{ post.title}}</h1>
      <p>{{ post.excerpt }}</p>
    </div>
    {% continue %}
  {% endif %}

  {% if pos_post >= size_post %}
    {% assign project = site.projects[pos_proj] %}
    {% assign pos_proj =  pos_proj | plus: 1 %}
    {% assign date = project.date | split: " " %}
    <div class="timeline-item" date-is='{{ date[0] }}'>
      <h1>{{ project.title}}</h1>
      <p>{{ project.excerpt }}</p>
    </div>
    {% continue %}
  {% endif %}

  {% assign project = site.projects[pos_proj] %}
  {% assign post = site.posts[pos_post] %}
  {% if project.date >= post.date %}
    {% assign pos_proj =  pos_proj | plus: 1 %}
    {% assign date = project.date | split: " " %}
    <div class="timeline-item" date-is='{{ date[0] }}'>
      <h1>{{ project.title}}</h1>
      <p>{{ project.excerpt }}</p>
    </div>
  {% else %}
    {% assign pos_post =  pos_post | plus: 1 %}
    {% assign date = post.date | split: " " %}
    <div class="timeline-item" date-is='{{ date[0] }}'>
      <h1>{{ post.title}}</h1>
      <p>{{ post.excerpt }}</p>
    </div>
  {% endif %}
{% endfor %}
</div>
