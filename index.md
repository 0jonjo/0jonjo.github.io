---
layout: home
permalink: /
---

## Latest posts
{% assign recent = site.posts | slice: 0, 5 %}
{% include post_list.html posts=recent %}

<p><a href="{{ "/blog/" | relative_url }}">All posts &raquo;</a> · <a href="{{ "/archive/" | relative_url }}">By year</a> · <a href="{{ "/tags/" | relative_url }}">By tag</a></p>
