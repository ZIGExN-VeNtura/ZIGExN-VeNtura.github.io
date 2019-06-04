---
layout: default
title: Post by categories
permalink: /categories
---
<div class="layout-page post-categories">
  <div class="section-title">
    <h2><span>Post by categories</span></h2>
  </div>
  <div class="row">
    {% for category in site.categories %}
      <div class="col-sm-3 category" onclick="location.href='#{{ category[0] | slugify }}';" style="cursor: pointer;">
        <a href="#{{ category[0] | slugify }}">
          <strong>{{ category[0] | capitalize }}</strong> <span class="pull-right">{{ category[1].size }}</span>
        </a>
        <hr>
      </div>
    {% endfor %}
  </div>
</div>
<section class="row categories">
  {% for category in site.categories %}
    <div class="col-md-12 col-lg-12">
      <div class="section-title">
        <h2 id="{{ category[0] | slugify }}" ><span>Category '{{ category[0] }}'</span></h2>
      </div>
      <div class="row listrecent">
        {% assign pages_list = category[1] %}
        {% for post in pages_list %}
        {% if post.title != null %}
        {% if group == null or group == post.group %}
          {% assign author = site.authors[post.author] %}
          <div class="col-sm-4 d-flex">
            <div class="card card-body flex-fill postbox">
              <div class="card-block">
                <div class="author-meta">
                  <span class="post-name"><a target="_blank" href="{{ site.base_url }}/authors">{{ author.name }}</a></span>
                  <span class="post-date">{{ post.date | date_to_string }}</span>
                </div>
                <h2 class="card-title"><a href="{{ post.url }}">{{ post.title }}</a></h2>
              </div>
            </div>
          </div>
        {% endif %}
        {% endif %}
        {% endfor %}
        {% assign pages_list = nil %}
        {% assign group = nil %}
      </div>
    </div>
  {% endfor %}
</section>
