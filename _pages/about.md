---
permalink: /
title: "The Time Line"
excerpt: "Time Line"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

<div class="container">
{% for post in site.projects reversed %}
  <div class="timeline-item" date-is='20-07-1990'>
    <h1>{{ post.title}}</h1>
    <p>{{ post.excerpt }}</p>
  </div>
{% endfor %}
</div>