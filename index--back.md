---
layout: home
---


{% for post in site.posts %}

<div class = "post-title"> 
  <a href="www.baidu.com">{{post.title}}</a> <br/> → {{post.date|date_to_string}}

</div>

<div class = "post-description"><b>摘要：</b><br/>
&nbsp;&nbsp;&nbsp;&nbsp;{{post.description}}<br/>

</div>

<div class="btn-readmore">
<a >阅读更多</a>
</div>

<br/>
{% endfor %}