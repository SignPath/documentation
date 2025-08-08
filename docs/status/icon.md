---
# renders the svg alone
---
{%- if site.data.status.current.type == "good" -%}
	{%- include status_good.svg -%}
{%- elsif site.data.status.current.type == "warning" -%}
	{%- include status_warning.svg -%}
{%- elsif site.data.status.current.type == "critical" -%}
	{%- include status_critical.svg -%}
{%- endif -%}