---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Directorio de contenido

A continuación se enumeran hipervínculos a cada uno de los ejercicios de laboratorio y demostraciones.

## Laboratorios

{% assign Exercise = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Módulo| Ejercicio |
| --- | --- | 
{% for activity in Exercise  %} | {{ activity.Exercise.module }} | [{{ activity.Exercise.title }}{% if activity.Exercise.type %} - {{ activity.Exercise.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

