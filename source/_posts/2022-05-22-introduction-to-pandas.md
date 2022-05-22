---
layout: post
title: 十分钟快速入门 Pandas
date: 2022-05-22
tags:
- Python
- Pandas
categories: Python
description: 通过带有标签的列和索引，Pandas 使我们可以以一种所有人都能理解的方式来处理数据。它可以让我们毫不费力地从诸如 csv 类型的文件中导入数据。我们可以用它快速地对数据进行复杂的转换和过滤等操作。Pandas 真是超级棒。
---

> Source: [十分钟快速入门 Pandas](https://zhuanlan.zhihu.com/p/21933466)

Pandas 是我最喜爱的库之一。通过带有标签的列和索引，Pandas 使我们可以以一种所有人都能理解的方式来处理数据。它可以让我们毫不费力地从诸如 csv 类型的文件中导入数据。我们可以用它快速地对数据进行复杂的转换和过滤等操作。Pandas 真是超级棒。

我觉得**它和 Numpy、Matplotlib 一起构成了一个 Python 数据探索和分析的强大基础**。Scipy （将会在下一篇推文里介绍）当然也是一大主力并且是一个绝对赞的库，但是我觉得前三者才是 Python 科学计算真正的顶梁柱。

那么，赶紧看看 python 科学计算系列的第三篇推文，一窥 Pandas 的芳容吧。如果你还没看其它几篇文章的话，别忘了去看看。

## 导入 Pandas

第一件事当然是请出我们的明星 —— Pandas。

```Python
import pandas as pd # This is the standard
```

这是导入 pandas 的标准方法。我们不想一直写 pandas 的全名，但是保证代码的简洁和避免命名冲突都很重要，所以折中使用 pd 。如果你去看别人使用 pandas 的代码，就会看到这种导入方式。

## Pandas 中的数据类型

Pandas 基于两种数据类型，`series`和`dataframe`。

- **series** 是一种一维的数据类型，其中的每个元素都有各自的标签。如果你之前看过这个系列关于Numpy 的推文，你可以把它当作一个由带标签的元素组成的 numpy 数组。标签可以是数字或者字符。

- **dataframe** 是一个二维的、表格型的数据结构。Pandas 的 dataframe 可以储存许多不同类型的数据，并且每个轴都有标签。你可以把它当作一个 series 的字典。

## 将数据导入 Pandas

在对数据进行修改、探索和分析之前，我们得先导入数据。多亏了 Pandas ，这比在 Numpy 中还要容易。

这里我鼓励你去找到自己感兴趣的数据并用来练习。你的（或者别的）国家的网站就是不错的数据源。如果要举例的话，首推[英国政府数据](https://data.gov.uk/data/search)和[美国政府数据](http://catalog.data.gov/dataset)。[Kaggle](https://www.kaggle.com/)也是个很好的数据源。

我将使用英国降雨数据，这个数据集可以很容易地从英国政府网站上下载到。此外，我还下载了一些日本降雨量的数据。
