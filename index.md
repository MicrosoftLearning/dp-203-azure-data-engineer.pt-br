---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

# Exercícios de Engenharia de Dados do Azure

Os exercícios a seguir dão suporte aos módulos de treinamento no Microsoft Learn que dão suporte à certificação [Microsoft Certified: Azure Data Engineer Associate](https://learn.microsoft.com/certifications/azure-data-engineer/).

Para concluir esses exercícios, você precisará de uma [assinatura do Microsoft Azure](https://azure.microsoft.com/free) na qual tenha acesso administrativo. Para alguns exercícios, talvez você também precise de acesso a um [locatário do Microsoft Power BI](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Exercício | No ILT, este é um... |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} — {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}
