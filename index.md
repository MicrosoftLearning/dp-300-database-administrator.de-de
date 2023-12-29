---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Übungen zur Datenbankverwaltung

Diese Übungen unterstützen den Microsoft-Kurs [DP-300: Verwalten von Microsoft Azure SQL-Lösungen](https://docs.microsoft.com/training/courses/dp-300t00).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Modul | Übung |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} – {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

