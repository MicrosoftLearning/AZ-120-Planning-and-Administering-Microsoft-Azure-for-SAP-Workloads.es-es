---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Directorio de contenido: AZ-120: Planificación y administración de Microsoft Azure para cargas de trabajo SAP

Los archivos de laboratorios necesarios se pueden [descargar aquí](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

A continuación se enumeran hipervínculos a cada uno de los ejercicios de laboratorio y demostraciones.

## Laboratorios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| Módulo | Laboratorio |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
