---
title: Status
header: Status
description: SignPath SaaS environment operational status
datasource: status
layout: status
---

<section class="status-section">
	<div>
		<h2 class='{{ site.data.status.current.type }}'>
			{%- if site.data.status.current.type == "critical" -%}
				{%- include status_critical.svg -%}
			{%- elsif site.data.status.current.type == "warning" -%}
				{%- include status_warning.svg -%}
			{%- elsif site.data.status.current.type == "good" -%}
				{%- include status_good.svg -%}
			{%- else -%}
				[unknown status type]
			{%- endif -%}
			{{ site.data.status.current.title }}
		</h2>

		<div>
			{{ site.data.status.current.description | newline_to_br }}
			{%- if site.data.status.current.link -%} <a href="{{ site.data.status.current.link }}"> (Read more)</a> {%- endif -%}
		</div>


		<h3>Planned maintenance</h3>
		{%- unless site.data.status.planned -%}
			<div>No maintenance planned.</div>
		{%- endunless -%}
		<ul>
			{%- for incident in site.data.status.planned -%}
				<li class="{{ incident.type }}">
					<h3>{{ incident.date}}</h3>
					<label>{{ incident.range }}</label>
					<h4>{{ incident.title }}</h4>
					<div markdown="1">
					
{{ incident.description }}
{% if incident.link %} <a href="{{ incident.link }}"> (Read more)</a> {% endif %}
</div>
				</li>
			{%- endfor -%}
		</ul>

		<h3>Past incidents and maintenance</h3>
		<ul>
			{%- for incident in site.data.status.incidents -%}
				<li class="{{ incident.type }}">
					<h3>{{ incident.date}}</h3>
					<label>{{ incident.range }}</label>
					<h4>{{ incident.title }}</h4>
					<div markdown="1">

{{ incident.description }}
{% if incident.link %} <a href="{{ incident.link }}"> (Read more)</a> {% endif %}
</div>
				</li>
			{%- endfor -%}
		</ul>
		<p>
			Note that system updates and SignPath software updates are not listed here unless they may result in more than 5 minutes of downtime.
			We recommend that you set a retry limit for <i>unavailable service</i> errors of 10 minutes. This is also the default for our <a href="/build-system-integration#powershell">PowerShell scripts</a>.
		</p>
	</div>
</section>

