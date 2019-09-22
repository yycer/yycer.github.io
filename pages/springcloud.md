---
layout: page
title: Spring Cloud系列文章
titlebar: Spring Cloud
menu: Spring Cloud
css: ['blog-page.css']
permalink: /springcloud
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='springcloud' or post.keywords contains 'springcloud' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                        <span class='circle'></span>
                    </div>
                </li>
                {% endif %}
            {% endfor %}
        </ul> 

        <!-- Pagination -->
        <!-- {% include pagination.html %} -->

        <!-- Comments -->
       <!-- <div class="comment">
         {% include comments.html %}
       </div> -->
    </div>

</div>
<script>
    $(document).ready(function(){

        // Enable bootstrap tooltip
        $("body").tooltip({ selector: '[data-toggle=tooltip]' });

    });
</script>