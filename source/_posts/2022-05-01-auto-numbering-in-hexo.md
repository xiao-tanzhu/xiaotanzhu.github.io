---
layout: post
title: 使用为文章标题自动编号
date: 2022-05-01
tags:
- Hexo
- CSS
categories: Hexo
description: 使用css自动为文章的H1-H6标题自动编号
---

## 问题描述

我们都知道，MarkDown可以通过`#` `##` `###`等标记添加不同层次的标题，非常方便。但是问题来了：如何给标题加上编号呢？

```markdown
# Title
## 1. sub-title
### 1.1 sub-sub-title
### 1.2 sub-sub-title
## 2. sub-title
## 3. sub-title
```

为了给标题加上编号，手动加上11.11.223，很麻烦有没有？如果插入或删除章节，后面的编号都要重新修改一遍

## 解决方法

CSS可以自动在某些元素前插入编号，而Hexo又支持自定义css样式。那么解决方法来了。

### 开启Hexo自定义样式

我使用hexo的next主题，在根目录的`//_config.next.yml`文件中增加以下几行：

> Hexo的自定义样式参考：https://theme-next.js.org/docs/advanced-settings/custom-files

> **注意：**以下配置从主题目录配置文件中拷贝：//themes/next/_config.yml，不修改主题目录下的文件。

```yml
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  #footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```

### 添加自定义css样式

在`//source/_data/styles.styl`中添加以下内容：
```CSS
body { counter-reset: h2counter; }
h2 { counter-reset: h3counter; }
h3 { counter-reset: h4counter; }
h4 { counter-reset: h5counter; }
h5 { counter-reset: h6counter; }
h6 { }

.post-body h2:before {
  counter-increment: h2counter;
  content: counter(h2counter) ".\0000a0\0000a0";
}
.post-body h3:before {
  counter-increment: h3counter;
  content: counter(h2counter) "."counter(h3counter) ".\0000a0\0000a0";
}
.post-body h4:before {
  counter-increment: h4counter;
  content: counter(h2counter) "."counter(h3counter) "."counter(h4counter) ".\0000a0\0000a0";
}
.post-body h5:before {
  counter-increment: h5counter;
  content: counter(h2counter) "."counter(h3counter) "."counter(h4counter) "."counter(h5counter) ".\0000a0\0000a0";
}
.post-body h6:before {
  counter-increment: h6counter;
  content: counter(h2counter) "."counter(h3counter) "."counter(h4counter) "."counter(h5counter) "."counter(h6counter) ".\0000a0\0000a0";
}
```

## 其他说明

个人不喜欢使用H1标题，觉得太大了，所以编号都是从2级标题开始的。如果有喜欢H1标题的，可以对应修改以上css文件。
