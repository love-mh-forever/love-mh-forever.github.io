---
layout: post
title: Sublime Text3 + Markdown + 实时预览
no-post-nav: true
category: other
tags: [other]
---
# Sublime Text3 + Markdown + 实时预览

* 找到菜单栏：```Preferences``` - > ```Package Control``` -> ```Package Control Package```;

### 安装过程
* 需要两款插件：```Markdown Editing``` + ```MarkdownLivePreview```；
* Package Control → Install 
* Package中输入两款插件的名字，找到相应插件，点击即可自动完成安装，安装完重启Sublime
* 简单设置：Preferences → Package Settings → MarkdownLivePreview → Setting，打开后将左边default的设置代码复制到右边User栏，找到"markdown_live_preview_on_open": false,把false改为true，保存。