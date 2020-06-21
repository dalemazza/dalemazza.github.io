---
layout: default
---

## Welcome to my Blog


### Click here to write-ups

## Click here for my blog

{% for post in site.posts %}	
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p><small><strong>{{ post.date | date: "%B %e, %Y" }}</strong> . {{ post.category }} . <a href="http://dalemazza.github.io{{ post.url }}#disqus_thread"></a></small></p>			
{% endfor %}	
