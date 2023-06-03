---
title: Thoughts
---

{% assign docs = site.docs | where: "categories","thoughts" | sort:"order" %}
{% for doc in docs %}
* [{{ doc.title }}]({{ doc.url }}): {{ doc.desc }}
{% endfor %}
