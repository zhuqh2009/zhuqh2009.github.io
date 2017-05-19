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
		  {% for category in site.categories %}
			<h2>{{ category | first }}</h2>
			</span>{{ category | last | size }}</span>
			<ul class="arc-list">
			    {% for post in category.last %}
				<li>{{ post.date | date:"%d/%m/%Y"}}<a href="{{ post.url }}">{{ post.title }}</a></li>
			    {% endfor %}
			</ul>
		 {% endfor %}
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
