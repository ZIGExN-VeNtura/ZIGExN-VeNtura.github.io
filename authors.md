---
layout: page
title: Authors
permalink: /authors
---

<div class="row team">
  {% assign posts_grouped = site.posts | group_by: "author" | sort: 'size' | reverse %}
  {% for author in posts_grouped %}
    {% assign author_info = site.authors[author.name] %}
    {% assign author_name = author_info.name | default: author.name %}
    {% assign author_img = author_info.img | default: "https://www.gravatar.com/avatar/?s=250&d=mm&r=x" %}
    <div class="author col-md-3">
      <a href="{{ author_info.url }}" target="_blank"><strong>{{ author_name }}</strong></a>
      <img src="{{ author_img }}"/>
      <small>{{ author.items.size }} posts</small>
    </div>
  {% endfor %}
</div>
