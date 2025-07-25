---
title: "友情链接"
permalink: /friends/
layout: collection
---

<div class="friend-page">


<div class="friend-intro">
  这里收录了朋友们的博客和一些常用网站的链接。
</div>

{% for section in site.data.friends %}
<h2 class="friend-category">{{ section.category }}</h2>
<div class="friend-grid">
  {% for friend in section.links %}
  <div class="friend-card">
    <div class="friend-header">
      <!-- 优先显示头像 -->
      {% if friend.avatar %}
      <img src="{{ friend.avatar }}" alt="{{ friend.name }}" class="friend-avatar">
      <!-- 没有头像则显示图标 -->
      {% elsif friend.icon %}
      <span class="friend-icon">
        <i class="{{ friend.icon }}"></i>
      </span>
      <!-- 都没有则显示默认图标 -->
      {% else %}
      <span class="friend-icon">
        <i class="fas fa-user"></i>
      </span>
      {% endif %}
      
      <a href="{{ friend.url }}" target="_blank" class="friend-name">
        {{ friend.name }}
      </a>
    </div>
    
    <div class="friend-desc">{{ friend.desc }}</div>
    
    <div class="friend-meta">
      {% if friend.date %}
      <span class="friend-date">
        <i class="fas fa-calendar-alt"></i> 添加于 {{ friend.date }}
      </span>
      {% endif %}
      
      <div class="friend-link">
        <a href="{{ friend.url }}" target="_blank">
          <i class="fas fa-external-link-alt"></i> 访问网站
        </a>
      </div>
    </div>
  </div>
  {% endfor %}
</div>
{% endfor %}


<div class="friend-apply">
  <h4>想要交换友链？</h4>
  <ul>
  <ul>
    <h4>想要推荐网站？</h4>
    <ul>
    <ul>
      <h4>想要文字交流？</h4>
      <ul>
      <ul>
        <p><a href="mailto:mokaluona@outlook.com">这里联系</a></p>
      </ul>
      </ul>
    </ul> 
    </ul>
  </ul>
  </ul>
</div>

</div>

<style>
/* ===== 整体页面样式 ===== */
.friend-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* 介绍文字样式 */
.friend-intro {
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-left: 4px solidrgb(99, 76, 175);
  padding: 20px;
  margin-bottom: 40px;
  border-radius: 0 8px 8px 0;
  font-size: 1.05em;
  line-height: 1.6;
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
}

/* 分类标题样式 */
.friend-category {
  position: relative;
  padding-left: 15px;
  margin-bottom: 25px;
  color: #2c3e50;
  font-weight: 600;
}

.friend-category::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 5px;
  height: 24px;
  background-color:rgb(76, 163, 175);
  border-radius: 3px;
}

/* ===== 网格布局 ===== */
.friend-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  margin-bottom: 50px;
}

/* ===== 卡片样式 ===== */
.friend-card {
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  padding: 15px 20px;
  transition: all 0.3s ease;
  background: white;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
  display: flex;
  flex-direction: column;
  height: 100%;
}

.friend-card:hover {
  transform: translateY(-7px);
  box-shadow: 0 12px 30px rgba(0,0,0,0.12);
  border-color:rgb(120, 76, 175);
}

/* ===== 卡片头部 ===== */
.friend-header {
  display: flex;
  align-items: center;
  margin-bottom: 18px;
}

/* 头像样式 */
.friend-avatar {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  margin-right: 15px;
  object-fit: cover;
  border: 2px solid #f5f5f5;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

/* 图标样式 */
.friend-icon {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 15px;
  background: linear-gradient(135deg, #e3f2fd, #bbdefb);
  border-radius: 50%;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.friend-icon i {
  font-size: 28px;
  color: #1976D2;
}

/* 名称样式 */
.friend-name {
  font-size: 1.25em;
  font-weight: 600;
  color: #333;
  text-decoration: none;
  transition: color 0.3s;
}

.friend-name:hover {
  color:rgb(155, 76, 175);
}

/* ===== 描述样式 ===== */
.friend-desc {
  color: #555;
  line-height: 1.6;
  margin-bottom: 20px 10px;
  flex-grow: 1;
  font-size: 0.95em;
}

/* ===== 元信息样式 ===== */
.friend-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 15px;
  border-top: 1px solid #f0f0f0;
}

.friend-date {
  color: #777;
  font-size: 0.85em;
  display: flex;
  align-items: center;
}

.friend-date i {
  margin-right: 5px;
  color: #9e9e9e;
}

.friend-link a {
  display: inline-flex;
  align-items: center;
  padding: 8px 16px;
  background: linear-gradient(135deg, #f0f7ff, #e1f0ff);
  color: #1976D2;
  border-radius: 6px;
  text-decoration: none;
  font-size: 0.9em;
  font-weight: 500;
  transition: all 0.3s;
  box-shadow: 0 2px 5px rgba(25, 118, 210, 0.15);
}

.friend-link a:hover {
  background: linear-gradient(135deg, #1976D2, #0d47a1);
  color: white;
  box-shadow: 0 4px 8px rgba(25, 118, 210, 0.3);
}

.friend-link i {
  margin-right: 6px;
}

/* ===== 申请区域 ===== */
.friend-apply {
  background: linear-gradient(135deg, #f9fbe7,rgb(195, 244, 233));
  border-radius: 12px;
  padding: 25px;
  margin-top: 30px;
  border: 1px dashed rgb(149, 202, 51);
} 

.friend-apply h4 {
  margin-top: 0;
  color: #827717;
  display: flex;
  align-items: center;
}

.friend-apply h4 i {
  margin-right: 10px;
}

.friend-apply ul {
  padding-left: 20px;
  margin: 15px 0;
}

.friend-apply li {
  margin-bottom: 8px;
}

/* ===== 响应式设计 ===== */
@media (max-width: 768px) {
  .friend-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  
  .friend-avatar, .friend-icon {
    width: 50px;
    height: 50px;
  }
  
  .friend-icon i {
    font-size: 22px;
  }
  
  .friend-meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
}

@media (max-width: 480px) {
  .friend-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .friend-avatar, .friend-icon {
    margin-right: 0;
    margin-bottom: 15px;
  }
}
</style>