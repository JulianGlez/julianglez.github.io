---
layout: page
title: Certificaciones
icon: fa-solid fa-certificate
order: 1
---

{% assign certifications = site.posts
| where_exp: "post", "post.categories contains 'Certificaciones'"
| sort: "date"
| reverse
%}

{% if certifications.size == 0 %}
<p class="text-muted">Aún no hay certificaciones publicadas.</p>
{% else %}

{% assign current_year = "" %}

{% for post in certifications %}
{% assign post_year = post.date | date: "%Y" %}

{% if post_year != current_year %}
{% unless forloop.first %}
</div>
{% endunless %}

<h2 class="cert-year">{{ post_year }}</h2>
<div class="cert-grid">

{% assign current_year = post_year %}
{% endif %}
<div>
<a class="cert-item">
{% if post.image %}
<img
src="{{ post.image | relative_url }}"
alt="{{ post.title }}"
title="{{ post.title }} ({{ post.date | date: '%m/%y' }})"
class="cert-post-image">
{% endif %}
<span><a href="{{ post.url | relative_url }}">{{ post.title }}</a></span>
</a>
</div>
{% if forloop.last %}
</div>
{% endif %}
{% endfor %}

{% endif %}
