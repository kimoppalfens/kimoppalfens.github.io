---
layout: archive
title: OSCC Password Solutions
permalink: /PasswordSolutions/
author: oscc
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
  <div class="list__item">
  <article class="archive__item" itemscope itemtype="http://schema.org/CreativeWork">
  <p><h2 class="archive__item-title" itemprop="headline"><a href="{{ post.url }}"><b>{{ post.title }}</b></a></h2></p>
  <p class="archive__item-excerpt" itemprop="description">{{ post.excerpt }}</p>
  </article>
  </div>
  {% endif %}
{% endfor %}