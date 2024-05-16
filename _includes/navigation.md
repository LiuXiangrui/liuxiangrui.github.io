{% for link in site.data.navigation.main %}
  {% if link.right %}
    <a class="normal right" href="{{ site.url }}/{{ link.url }}">{{ link.title }}</a>
    {% else %}
    <a class="normal" href="{{ site.url }}/{{ link.url }}">{{ link.title }}</a>
  {% endif %}
{% endfor %}

