---
layout: page
title: About
permalink: /about/
---

{{site.description}}

Here are some technologies i am into:

<ul>
{% for key_word in site.key_words %}
  <li>{{key_word}}</li>
{% endfor %}
</ul>
