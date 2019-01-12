我的年度总结系列：

<ul>
    {% for post in site.tags["Annual Review"] %}{% if post.title != null %}
    <li class="entry-title"><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></li>
    {% endif %}{% endfor %}
</ul>
