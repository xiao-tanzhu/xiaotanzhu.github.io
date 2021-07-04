---
layout: post
title: 自动为Hexo博客添加文章链接
date: 2021-07-04
tags: ["Hexo", "Post Links", "Github Pages"]
categories: Hexo
description: 为Hexo博客添加文章链接，或文章版权说明等
---

文章链接是写博客时一个非常常用的东西，因为如果每次写博客都需要在文章前后加上一个几乎格式一模一样的东西，还是很烦人的，特别如果你突然决定改变文章的永久链接，那么就更加烦人了，因为默认在Hexo里面，我们需要一篇文章一篇文章的把所有的文章链接改了（虽然这种情况我们需要尽量避免）。

这里我们介绍一个插件，可以自动为博客添加文章链接：

> hexo-tag-post-link

项目地址：https://github.com/r12f/hexo-tag-post-link

## 安装

和其他的hexo插件一样，hexo-tag-post-link安装起来十分简单，只需要在博客目录下执行如下命令即可：

```bash
npm install hexo-tag-post-link --save
```

## 使用方法

### 配置模板

#### 添加模板文件：post_link.yml

1. 首先我们需要在`source`目录下创建一个`_data`目录，如果没有的话。`source/_data`这个目录是Hexo的数据目录，用来存放一些公用的全局数据，所以的模板文件也放在这里。
1. 现在在`_data`目录下创建一个名为`post_link.yml`的文件，这个文件就是我们的配置文件了。
1. 现在我们可以来添加模板了！模板的格式非常简单，如下：

```
name: format
```

比如我的博客使用的配置如下，而最后的效果就如本文最上方显示的那样。

```html
footer: <br><hr><b>转载请注明出处：</b><a href="<%= post_permalink %>" target="_blank"><%= post_title %></a><br><b>原文地址：</b><%= post_permalink %>
```

#### 可用模板变量

- site_title
- site_subtitle
- site_description
- site_author
- site_url
- post_title
- post_slug
- post_created
- post_created_date
- post_created_time
- post_updated
- post_updated_date
- post_updated_time
- post_relative_url
- post_permalink

### 增加全局配置

在配置文件_config.yml中添加如下配置：
```yml
post_link:
  #insert_before_post: header
  insert_after_post: footer
```

以上配置完成，即可在文章末尾看到文章声明和原文地址。

## 更多使用方法

可参考Github中的项目说明：https://github.com/r12f/hexo-tag-post-link
