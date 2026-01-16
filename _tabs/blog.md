---
layout: default
title: Blog
icon: fa-solid fa-newspaper
order: 2
---

# Publicaciones

<div class="row">
  <div id="core-wrapper" class="col-12 col-lg-11 col-xl-9 px-md-4" style="width:100%">
    
    <div id="post-list" class="flex-grow-1 px-xl-1">
      
      {% assign blog_posts = site.categories.Blog | default: site.empty_array %}
      {% if blog_posts.size > 0 %}
        {% assign blog_posts = blog_posts | sort: "date" | reverse %}
        {% assign pinned_posts = blog_posts | where: "pin", true %}
        {% assign normal_posts = blog_posts | reject: "pin", true %}
        {% assign posts = pinned_posts | concat: normal_posts %}
      {% endif %}

      {% for post in posts %}
        <article class="card-wrapper card">
          <a href="{{ post.url | relative_url }}" class="post-preview row g-0 flex-md-row-reverse">
            {% assign card_body_col = '12' %}

            {% if post.image %}
              {% assign src = post.image.path | default: post.image %}

              {% if post.media_subpath %}
                {% unless src contains '://' %}
                  {% assign src = post.media_subpath | append: '/' | append: src | replace: '///', '/' | replace: '//', '/' %}
                {% endunless %}
              {% endif %}

              {% if post.image.lqip %}
                {% assign lqip = post.image.lqip %}
                {% if post.media_subpath %}
                  {% unless lqip contains 'data:' %}
                    {% assign lqip = post.media_subpath | append: '/' | append: lqip | replace: '///', '/' | replace: '//', '/' %}
                  {% endunless %}
                {% endif %}
                {% assign lqip_attr = 'lqip="' | append: lqip | append: '"' %}
              {% endif %}

              {% assign alt = post.image.alt | xml_escape | default: 'Preview Image' %}

              <div class="col-md-5">
                <img src="{{ src }}" alt="{{ alt }}" {{ lqip_attr }}>
              </div>

              {% assign card_body_col = '7' %}
            {% endif %}

            <div class="col-md-{{ card_body_col }}">
              <div class="card-body d-flex flex-column">
                <h1 class="card-title my-2 mt-md-0">{{ post.title }}</h1>

                <div class="card-text content mt-0 mb-3">
                  <p>
                    {% include post-summary.html %}
                  </p>
                </div>

                <div class="post-meta flex-grow-1 d-flex align-items-end">
                  <div class="me-auto">
                    <i class="far fa-calendar fa-fw me-1"></i>
                    {% include datetime.html date=post.date lang=lang %}

                    {% if post.categories.size > 0 %}
                      <i class="far fa-folder-open fa-fw me-1"></i>
                      <span class="categories">
                        {{ post.categories | join: ", " }}
                      </span>
                    {% endif %}
                  </div>

                  {% if post.pin %}
                    <div class="pin ms-1">
                      <i class="fas fa-thumbtack fa-fw"></i>
                      <span>{{ site.data.locales[lang].post.pin_prompt }}</span>
                    </div>
                  {% endif %}
                </div>
              </div>
            </div>
          </a>
        </article>
      {% endfor %}
    </div>
  </div>

  <div id="panel-wrapper" class="col-xl-3 ps-2 text-muted">
    </div>
</div>
