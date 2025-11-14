Tailwind CSS + Hotwire 快速上手（Rails 版）
Tailwind CSS + Hotwire 快速上手手册（Rails 开发者版）

目录

1. [技术栈概述](#1-技术栈概述)
2. [环境准备](#2-环境准备)
3. [Tailwind CSS 核心用法](#3-tailwind-css-核心用法)
4. [Hotwire 实战指南](#4-hotwire-实战指南)
5. [实战项目：迷你知识库](#5-实战项目迷你知识库)
6. [学习资源与代码仓库](#6-学习资源与代码仓库)

1. 技术栈概述

针对岗位要求的核心工具链，聚焦「后端开发者友好型」前端技术：

- Tailwind CSS：原子化CSS框架，用类名直接写样式，无需写传统CSS文件
- Hotwire：Rails官方推荐的前端交互方案，包含：
  - Turbo：通过HTML重载实现局部更新（替代AJAX）
  - Stimulus：轻量JS框架，处理DOM交互（语法接近Ruby）
- Git/GitLab：代码管理与协作流程（重点掌握MR操作）

2. 环境准备

2.1 集成到Rails项目

# 新建Rails项目（或在现有项目中执行）
rails new knowledge_base -j esbuild
cd knowledge_base

# 安装Tailwind CSS
bundle add tailwindcss-rails
rails tailwindcss:install

# 安装Hotwire（Rails 7+默认集成，如需手动安装）
bundle add turbo-rails stimulus-rails
rails turbo:install stimulus:install

# 启动服务器
bin/dev  # 同时运行Rails服务器和Tailwind编译器

2.2 验证安装

- 访问 http://localhost:3000，页面正常加载
- 检查 app/assets/stylesheets/application.tailwind.css 是否存在（Tailwind入口文件）
- 检查 app/javascript/controllers/ 是否存在（Stimulus控制器目录）

3. Tailwind CSS 核心用法

3.1 基础语法：工具类

直接在HTML标签中添加类名实现样式：

<!-- 示例：带样式的按钮 -->
<button class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 transition">
  保存文档
</button>

- bg-blue-600：背景色
- px-4 py-2：水平/垂直内边距
- rounded：圆角
- hover:bg-blue-700：鼠标悬停效果
- transition：平滑过渡动画

3.2 响应式设计

用前缀实现多设备适配（移动端优先）：

<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- 移动端1列，平板2列，桌面3列 -->
  <% @documents.each do |doc| %>
    <div class="border p-4"><%= doc.title %></div>
  <% end %>
</div>

- 前缀：sm:（640px+）、md:（768px+）、lg:（1024px+）

3.3 常用操作

- 自定义主题：修改 config/tailwind.config.js
  module.exports = {
  theme: {
  extend: {
  colors: {
  primary: '#3b82f6', // 项目主色调
  },
  fontFamily: {
  sans: ['Inter', 'system-ui'],
  },
  },
  }
  }
- 提取组件：用Rails Partial复用样式
<!-- app/views/shared/_btn.html.erb -->
<button class="bg-primary text-white px-4 py-2 rounded hover:bg-primary/90 <%= classes %>">
  <%= text %>
</button>

<!-- 使用时 -->
<%= render 'shared/btn', text: '删除', classes: 'bg-red-500' %>

4. Hotwire 实战指南

4.1 Turbo Frames：局部刷新

场景：表单提交后只更新页面部分区域

<!-- app/views/documents/new.html.erb -->
<turbo-frame id="document_form">
  <%= form_with model: @document do |form| %>
    <%= form.text_field :title, class: "border p-2" %>
    <%= form.submit "保存", class: "bg-primary text-white px-4 py-2" %>
  <% end %>
</turbo-frame>

<!-- app/views/documents/create.turbo_stream.erb -->
<turbo-stream action="replace" target="document_form">
  <template>
    <p class="text-green-600">文档创建成功！</p>
  </template>
</turbo-stream>

4.2 Turbo Streams：实时更新

场景：新评论提交后，所有页面自动显示新评论

<!-- app/views/comments/create.turbo_stream.erb -->
<turbo-stream action="append" target="comments">
  <template>
    <div class="border p-3 mt-2"><%= @comment.content %></div>
  </template>
</turbo-stream>

<!-- 在显示评论的页面添加 -->
<div id="comments">
  <%= render @document.comments %>
</div>

4.3 Stimulus：交互逻辑

场景：实现「点击展开/折叠文档内容」

1. 创建控制器：

// app/javascript/controllers/collapse_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
static targets = ["content"]

toggle() {
this.contentTarget.classList.toggle("hidden")
}
}

2. 在视图中使用：

<div data-controller="collapse">
  <button data-action="click->collapse#toggle" class="text-blue-600">
    点击展开
  </button>
  <div data-collapse-target="content" class="hidden mt-2">
    <%= @document.content %>
  </div>
</div>

5. 实战项目：迷你知识库

5.1 功能规划

- 文档CRUD（标题、内容、分类）
- 响应式列表页（移动端/桌面端适配）
- 实时评论功能（提交后无需刷新可见）
- 文档内容折叠/展开交互

5.2 核心代码实现

1. 模型与路由

rails g scaffold Document title:string content:text category:string
rails g model Comment content:text document:references
rails db:migrate

2. 列表页（Tailwind布局）

<!-- app/views/documents/index.html.erb -->
<div class="container mx-auto px-4 py-8">
  <h1 class="text-2xl font-bold mb-6">知识库</h1>

  <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
    <% @documents.each do |doc| %>
      <div class="border rounded-lg p-5 shadow-sm hover:shadow">
        <h2 class="text-xl font-semibold"><%= doc.title %></h2>
        <p class="text-gray-600 text-sm mt-1"><%= doc.category %></p>

        <!-- 折叠功能（Stimulus） -->
        <div data-controller="collapse" class="mt-3">
          <button data-action="click->collapse#toggle" class="text-blue-600 text-sm">
            查看内容
          </button>
          <div data-collapse-target="content" class="hidden mt-2 text-gray-700">
            <%= doc.content %>
          </div>
        </div>

        <!-- 评论区（Turbo Streams） -->
        <div class="mt-4">
          <h3 class="font-medium">评论</h3>
          <div id="comments_<%= doc.id %>">
            <%= render doc.comments %>
          </div>
          <%= form_with model: doc.comments.build, data: { turbo_stream: true } do |f| %>
            <%= f.text_field :content, class: "border p-2 w-full mt-2" %>
            <%= f.submit "提交", class: "bg-primary text-white px-3 py-1 mt-1" %>
          <% end %>
        </div>
      </div>
    <% end %>
  </div>
</div>

3. Git协作流程

# 创建分支
git checkout -b feature/add-knowledge-base

# 提交代码
git add .
git commit -m "实现文档列表和评论功能"

# 推送到远程（模拟GitLab）
git push -u origin feature/add-knowledge-base
# 然后在GitLab上创建MR（合并请求）

6. 学习资源与代码仓库

官方文档

- Tailwind CSS 官方文档
- Hotwire 官方指南
- Rails + Hotwire 集成文档
- GitLab 协作流程

推荐代码仓库

1. Tailwind + Rails 示例：dhh/tailwindcss-rails-demo（Rails作者实战）
2. Hotwire 实时功能：gorails/hotwire-demo-chat（聊天功能，类比评论系统）
3. 知识库参考：documentcloud/annotator（文档交互功能）
4. 组件库：excid3/tailwindcss-stimulus-components（现成组件可复用）

备注

- 学习优先级：Tailwind工具类 → Turbo Frames → Stimulus基础 → Turbo Streams
- 无需深入JS/CSS理论，聚焦「用Rails思维实现前端功能」
- 遇到问题先查官方文档，再参考推荐仓库的源码实现


---

可根据实际需求扩展章节，如需某部分更详细的代码示例（如Turbo Streams完整流程），可随时补充！