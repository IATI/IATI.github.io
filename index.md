---
layout: default
---

<div class="posts">
    {% for post in site.posts limit:10 %}
        <article class="post">

            <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>

            <p class="post-author">By {{ post.author }}</p>
            <p class="post-date"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%B %d, %Y" }}</time></p>


            <div class="entry">
                {{ post.excerpt }}
            </div>

            <a href="{{ post.url }}" class="read-more">Read More</a>
        </article>
        {% if forloop.index == 10 %}
            <a href="/blog/" class="read-more">Find more posts in the blog archive</a>
        {% endif %}
    {% endfor %}
</div>