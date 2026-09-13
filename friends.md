---
title: "友情链接"
permalink: /friends/
layout: archive
---

<div class="friend-page">
  <p class="friend-intro">这里收录了朋友们的博客和一些常用网站的链接。</p>

  {% for section in site.data.friends %}
    <h2 class="archive__subtitle">{{ section.category }}</h2>
    <div class="friend-grid">
      {% for friend in section.links %}
        <div class="friend-card">
          {% if friend.avatar %}
            <span class="friend-avatar"><img src="{{ friend.avatar }}" alt="{{ friend.name }}"></span>
          {% elsif friend.icon %}
            <span class="friend-icon"><i class="{{ friend.icon }}"></i></span>
          {% else %}
            <span class="friend-icon"><i class="fas fa-user"></i></span>
          {% endif %}
          <div class="friend-body">
            <a class="friend-name" href="{{ friend.url }}" target="_blank" rel="noopener">{{ friend.name }}</a>
            <p class="friend-desc">{{ friend.desc }}</p>
            {% if friend.date %}<span class="friend-date">添加于 {{ friend.date }}</span>{% endif %}
          </div>
        </div>
      {% endfor %}
    </div>
  {% endfor %}

  <div class="friend-apply">
    <p>想要交换友链、推荐网站，或只是文字交流？欢迎 <a href="mailto:mokaluona@outlook.com">邮件联系</a>。</p>
  </div>
</div>
