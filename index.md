---
title: Exercícios para desenvolvedores do Azure
permalink: index.html
layout: home
---

## Visão geral

Os exercícios a seguir foram projetados para proporcionar uma experiência de aprendizado prática na qual você explorará tarefas comuns que os desenvolvedores executam ao criar e implantar soluções para o Microsoft Azure.

> **Observação**: para fazer os exercícios, você precisará de uma assinatura do Azure na qual tenha permissões e cotas suficientes para provisionar os recursos do Azure necessários. Caso ainda não tenha uma conta do Azure, inscreva-se para obter uma [conta do Azure](https://azure.microsoft.com/free). 

Alguns exercícios podem ter requisitos adicionais ou diferentes. Eles conterão uma seção **Antes de começar** específica para cada exercício.

## Áreas de tópico
{% assign exercises = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign grouped_exercises = exercises | group_by: "lab.topic" %}

<ul>
{% for group in grouped_exercises %}
<li><a href="#{{ group.name | slugify }}">{{ group.name }}</a></li>
{% endfor %}
</ul>

{% for group in grouped_exercises %}

## <a id="{{ group.name | slugify }}"></a>{{ group.name }} 

{% for activity in group.items %} [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }}

---

{% endfor %} <a href="#overview">Voltar ao topo</a> {% endfor %}

