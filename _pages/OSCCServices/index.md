---
layout: archive
permalink: /osccservices/
title: "OSCC Services"
date: 2014-06-02T12:26:34-04:00
modified: 2015-12-02T11:05:08-05:00
excerpt: "A collection of thoughts, inspiration, mistakes, and other minutia I’ve written."
feature:
  visible: true
  headline: "OSCC Services"
  category: osccservices
---

{% for post in site.categories.osccservices %}
  {% if post.featured != true %}
  <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
  {% endif %}
{% endfor %}