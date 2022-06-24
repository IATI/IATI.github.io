---
layout: default
title: Code examples
permalink: /code/
---

Here you'll find a small selection of code examples written to make use of IATI's APIs, data, and tools. Unless otherwise noted, all code is licensed under [GNU Affero General Public License](https://www.gnu.org/licenses/agpl-3.0.en.html). In other words, you are free to modify, distribute, warranty, or commercially use this code as long as it's also licensed under the GNU Affero GPL. If you have an idea for a code example to host here, or want to see an existing example written in a different programming language, please let us know at [code@iatistandard.org](mailto:code@iatistandard.org).

Our code examples include:

<ul>
{% for page in site.pages  %}
{% if page.layout == "code" %}
<li><a href="{{ page.url }}">{{ page.title }}</a></li>
{% endif %}
{% endfor %}
</ul>

