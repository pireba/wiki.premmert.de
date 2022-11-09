---
title: 'Debian 11'
taxonomy:
    category:
        - docs
process:
    markdown: true
    twig: true
twig_first: true
---

{% macro recursiveTOC(pages) %}
	<ul>
		{% for page in pages %}
			{% set items = toc_items(page.content) %}
			<li>
				<a href="{{ page.url(true) }}">{{ page.title }}</a>
				{{ _self.recursiveTOCLoop(page.url(true), items) }}
			</li>
		{% endfor %}
	</ul>
{% endmacro %}

{% macro recursiveTOCLoop(baseURL, items) %}
	<ul>
		{% for item in items %}
			<li>
				<a href="{{ baseURL }}{{ item.uri }}">{{ item.label }}</a>
				{{ _self.recursiveTOCLoop(baseURL, item.children) }}
			</li>
		{% endfor %}
	</ul>
{% endmacro %}

<div class="toc">
	{% if config.plugins['page-toc'].enabled %}
		{{ _self.recursiveTOC(page.children) }}
	{% else %}
		Das Plugin "page-toc" ist nicht aktiv.
	{% endif %}
</div>