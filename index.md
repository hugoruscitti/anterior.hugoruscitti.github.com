---
layout: page
title: Bienvenido
---
{% include JB/setup %}

Este es un sitio de prueba.


## Publicaciones mas recientes:

{% assign posts = site.posts %}
{% assign listing_limit = 3 %}
{% include post-listing.html %}
