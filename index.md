---
layout: default
titile: 好记性不如烂笔头
---

<body>
  <div class="index-wrapper">
    <div class="aside">
      <div class="info-card">
        <h1>ioend</h1>
		<ul>
		    {\% for category in site.categories %}
		    <li><a href="/categories/{{ category | first }}/" title="view all
		posts">{{ category | first }} {{ category | last | size }}</a>
		    </li>
		    {\% endfor %}
		</ul>
      </div>
      <div id="particles-js"></div>
    </div>

    <div class="index-content">
      <ul class="artical-list">
        {% for post in site.categories.blog %}
        <li>
          <a href="{{ post.url }}" class="title">{{ post.title }}</a>
          <div class="title-desc">{{ post.description }}</div>
        </li>
        {% endfor %}
      </ul>
    </div>
  </div>
</body>
