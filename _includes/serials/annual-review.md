我的年终总结系列：

<ul>
    {% for post in site.tags["年终总结"] %}{% if post.title != null %}
    <li class="entry-title"><a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></li>
    {% endif %}{% endfor %}
</ul>
