---
layout: main
---

# Documents
{% for post in site.posts %}

### {{ post.date | date: "%Y/%m/%d" }} [{{ post.title | escape }}]({{ post.url | prepend: site.baseurl }})

{% endfor %}

<br>
# Friend Links

+ [Cyorage's Blog](https://ohayo.cyoragex.xyz/)


