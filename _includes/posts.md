<h2 id="posts" style="margin: 2px 0px 0px;">Posts</h2>

<div class="posts">
<ol class="post_content">


<!-- Print the contents of the link object -->
{% for post in site.posts %}
<li>
<div class="pub-row">
  <div class="col-sm-9" style="position: relative;padding-right: 15px;padding-left: 20px;">
      <div class="title"><a href="{{ post.url }}">{{ post.title }}</a></div>
  </div>
</div>
</li>
{% endfor %}

<br>

</ol>
</div>