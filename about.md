---
layout: page
permalink: /about/index.html
title: Ilyes Houdjedje
tags: [Ilyes, Author]
chart: true
---

<!-- [<img src="/images/my-avatar.jpeg" class="notepad-post-author-avatar" alt="ilyes's photo" style="height: 20%;width: 20%;"/>](/images/my-avatar.jpeg) -->


{% assign total_words = 0 %}
{% assign total_readtime = 0 %}
{% assign featuredcount = 0 %}
{% assign statuscount = 0 %}

{% for post in site.posts %}
    {% assign post_words = post.content | strip_html | number_of_words %}
    {% assign readtime = post_words | append: '.0' | divided_by:200 %}
    {% assign total_words = total_words | plus: post_words %}
    {% assign total_readtime = total_readtime | plus: readtime %}
    {% if post.featured %}
    {% assign featuredcount = featuredcount | plus: 1 %}
    {% endif %}
{% endfor %}

Hey, i'm **Ilyes**. and this is my story...
I started working with computer since I was 7, hence I fell in love so early. I started my coding journey in college at 19, it was a bit weird and hard at first, but working on side projects and homework gave me that self-confidence to build software and to get my hands dirty. After moving to Europe, I got more chance to get in touch with the industry and to sharpen my skill set. Thus, I pursued my journey in Data Engineering to take it to the next level. I may not be the smartest and I'm _maybe_ not the best **BUT**, here's the thing...I work the sh** out to get whatever I plan to and you know what!…it's all about DISCIPLINE, CONSISTENCY, and HARD WORK.

<!-- This is my personal blog and currently has {{ site.posts | size }} posts in {{ site.categories | size }} categories which combinedly have {{ total_words }} words, which will take an average reader ({{ site.wpm }} WPM) approximately <span class="time">{{ total_readtime }}</span> minutes to read. {% if featuredcount != 0 %}There are <a href="{{ site.url }}/featured">{{ featuredcount }} featured posts</a>, you should definitely check those out.{% endif %} The most recent post is {% for post in site.posts limit:1 %}{% if post.description %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}">"{{ post.title }}"</a>{% else %}<a href="{{ site.url }}{{ post.url }}" title="{{ post.description }}" title="Read more about {{ post.title }}">"{{ post.title }}"</a>{% endif %}{% endfor %} which was published on {% for post in site.posts limit:1 %}{% assign modifiedtime = post.modified | date: "%Y%m%d" %}{% assign posttime = post.date | date: "%Y%m%d" %}<time datetime="{{ post.date | date_to_xmlschema }}" class="post-time">{{ post.date | date: "%d %b %Y" }}</time>{% if post.modified %}{% if modifiedtime != posttime %} and last modified on <time datetime="{{ post.modified | date: "%Y-%m-%d" }}" itemprop="dateModified">{{ post.modified | date: "%d %b %Y" }}</time>{% endif %}{% endif %}{% endfor %}. The last commit was on {{ site.time | date: "%A, %d %b %Y" }} at {{ site.time | date: "%I:%M %p" }} [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time "Temps Universel Coordonné"). -->
