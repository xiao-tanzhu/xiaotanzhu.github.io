---
layout: post
title: 做产品为什么要画这些图？
date: 2021-07-10
tags: ["UML", "产品经理"]
categories: 产品经理
description: 做产品为什么要画这些图？
---

经常看到网上有人问，产品经理要画哪些图，怎么画流程图等关于画图的问题。

确实，画图是产品经理必备的硬核技能。然而，画图又不仅仅是画几个图而已。

做产品没有统一、标准的规范指导，容易让人为了画图而画图。

甚至，在很多外行看来，感觉产品经理就是个画图的。

还有人戏称，去大厂当产品经理，就是个画图仔。

幸好，我从最初做项目，就学着用 UML 进行需求分析。受益于此，我渐渐体会到，画图背后的逻辑和意义。

趁这段时间，从画图入手，用自己的理解，总结自己做产品的方法。

## 当我们画图时，是在画什么

### 探寻本源

在需求分析领域，谈到画图，不能不提 UML 。

早在IT软件时代，UML 就是一套非常好用的需求分析、系统分析设计的工具和方法。

**那么，什么是 UML ？**

UML（Unified Modeling Language）统一建模语言，是一种用来对软件系统开发的产出进行可视化、规范定义、构造和文档化的面向对象的标准建模语言。这是比较官方的定义。

我理解，它是用一套规范的可视化图形，及建模方法，来描述软件系统的分析、设计等各个阶段，最终形成可视化、文档化的产出。

作为一门语言，UML 在建模过程使用的基本图形元素，就是它的 “ 词汇 ” ，如用例、参与者、类等；而这些元素间的使用规则，好比 “ 语法 ” ，如用例图、活动图便是在这种 “ 语法 ” 指导下绘制的视图。

学习一门语言，既要学习其 “ 词汇 ” ，也要掌握其 “ 语法 ” 的运用，将 “ 词汇 ” 合理组织起来，才能编写足以 “ 传情达意 ” 的 “ 文章 ” 。

UML 中，通过元素、视图建立起来的模型，正是这样一种可准确描述需求，又便于用开发思维去理解的 “ 文章 ” 。

UML 不仅提供了一套工具，还提供了一套思想和方法。

学工具，只能让我们知其然；掌握该工具背后的思想和方法，才能让我们知其所以然、活学活用。

UML 中，将可视化视图分为 “ 静态视图 ” 和 “ 动态视图 ” 。从“ 静 ”和“ 动 ”两个维度，来描述软件系统。

这也是人们描述现实事物的方法。从 “ 静 ” 的方面，描述事物的结构性特征；从 “ 动 ” 的方面，要描述事物的行为性特征。

一静一动相结合，便能把事物全面、准确地描述清楚。

这让我明白，做产品需求分析，也是对需求进行描述和表达。

使用这种方法，从静态、动态两个视角维度去分析和描述需求，就有了指导思想，能提高准确性和效率。

### 画图的实质

这就不难理解，**当我们画图时，一方面是在借助工具，对业务、需求、场景等进行梳理；一方面是在对需求、产品进行描述，并输出可视化的材料，供相关人员，阅读使用。**

因此，画图，是需求分析的重要组成部分，是用可视化的方式，对需求进行梳理和展示。

产出的可视化图形，是后续产品规划、研发、设计、测试，及优化迭代、问题排查等的重要依据。

### 本文思路

UML 中的图形、视图、建模方法等知识内容很多，要深入学习，推荐《 大象：Thinking in UML 》一书。

本文，结合在产品工作用到的内容，及自己的理解、体会，先从整体进行总结，建立一个整体的认知框架，后续再具体深入。

为了方便理解，我结合在资讯、充值等方面的经验，虚构了一个小型 APP 的前端需求，作为案例来分析。从静态、动态两个维度，总结在产品工作中常用的视图。

![](/images/0007.jpeg)

## 从静态视角看需求

**从静态的视角去分析需求，是用静态视图，来描述产品的结构性特征，这些结构决定了产品是由什么组成、能做什么、长什么样的。**

在 UML 中，静态视图包括用例图、类图、包图。类图与包图，在产品层面较少使用，开发层面更为需要。

结合实际工作，总结产品的静态视图有：**结构图、用例图、无交互的原型图**。

### 结构图

结构图，顾名思义就是描述、展示产品的基本结构、框架。它能清晰展示产品有哪些模块、功能或系统组成，它们之间的层级、从属等结构关系是怎样的。

这在需求分析的初始阶段，就要明确。

好比，建一栋楼，需要有建设蓝图，得先知道楼要建多高、地基要挖多深、面积多大、盖多少层等基本的框架信息。

结构图，像一栋楼的框架，那么一旦确立，就很难改，除非推倒重来。所以，一开始一定要搞清楚，避免挖坑。

当然，在基本框架下，随着对需求的不断深挖，各模块、功能、系统之间的结构、层级、从属关系，会更加清晰，结构图也要细化、完善。

根据使用情况，结构图大致可分为：**功能结构图、信息结构图、产品结构图**。

#### 功能结构图

功能结构图，就是描述产品都有哪些功能，层级和归属关系是怎样的。

这在产品工作中，经常用到。大部分人做产品，都知道在规划产品时，要画功能结构图。

现在用案例来看看这些图是怎么画的。

这个虚构 APP 的需求是：要做一个 APP ，给用户提供电商优惠活动信息、手机话费充值，后续会加更多功能，发展用户体系等。

一般经过需求调研和确认，脑海里就要有个大概的框架，这个 APP 给什么人用的、由哪些功能组成。

这个需求，我只能先脑补需求调研和确认的过程，然后得出这个产品大概要有：首页聚合展示、优惠活动、话费充值、个人中心，这四个大的功能模块。

![功能结构图](/images/0008.jpeg)

#### 信息结构图

信息架构图，跟功能结构图，容易搞混。

其实，有所不同，**信息结构图重点在 “ 信息 ”**。也就是，从信息的维度，将整个产品的信息进行抽象、归类，说明产品包括哪些信息、字段数据。

这有点类似，UML 中的类图。在产品层面，更多是要把产品里面的信息进行整理、归类、描述清楚，特别是信息的类型、条件、规则。比如，标题字数上限、图片尺寸等。

案例中的需求进行抽象，可以得出如下简单的信息结构图。

![信息结构图](/images/0022.png)

#### 产品结构图

产品结构图，网上说法不一，这本来就没什么标准，关键是看怎么理解、怎么用，能把需求准确、清晰地描述出来即可。

个人理解，产品结构图，就是「 功能结构图 」与「 信息结构图 」的结合。

就是在描述功能结构后，把对应功能模块有哪些信息，这些信息字段的定义、规则、条件、类型都列出来即可。

这种做法，更为常用。这个图画出来内容比较多，因为篇（太）幅（懒）关系，这里就不再画，可以自行脑补下。

### 用例图

用例图，采用参与者和用例来展现产品的功能性需求，是 UML 中一种很重要的视图。

在 UML 中，将用例图，分为业务用例图和系统用例图。

除了画图，还要写用例规约，以描述说明用例，包括对用例的描述、参与者、前后置条件、基本流程等，一般用表格形式比较清晰。

#### 业务用例图

业务用例图，是**从业务的视角，通过业务建模，对业务进行描述**。

这里说的业务，是在还没有新系统或产品前，业务是如何进行的。

UML 使用业务用例图，进行业务建模，旨在把业务描述清楚，发现业务问题和难点，这样的新系统才能更好的融入业务去解决问题。

简单的需求，很少画业务用例图；如面对比较复杂、有规模的需求，建议最好用这个思路进行分析，可以更好地发现业务问题，以保证产品需求不跑偏。

案例需求中，就话费充值这个业务，可画成业务用例图如下。

![业务用例图](/images/0009.jpeg)

#### 系统用例图

系统用例图，是对有新系统或产品后，业务又是如何进行的，进行建模描述。它是对业务用例进行分析得到的，是系统的开发范围。

简单看来，「 系统用例图」与「 功能结构图」很类似。

功能结构图，只是从产品或系统层面，描述都有哪些功能。

**系统用例图，则是从使用者的角度，描述对应用户能使用产品做什么**。这样的好处，是让我们时刻以用户为中心，思考产品和功能。

好多人，在做产品时，经常做着做着就忘了用户是谁、产品或功能是给谁用的。使用系统用例图，可以帮我们避免这一点。

以下为案例需求的系统用例图。

![系统用例图](/images/0010.jpeg)

#### 原型图（无交互）

原型图，是产品经理最熟悉不过的，也是最常用的。甚至，有不少人，以为做产品就是画原型图，一接到需求，巴不得马上打开 Axure 画图。

众所周知，原型图，是产品表现层面的 Demo ，描绘产品的界面长什么样，功能如何设计、摆放，有哪些内容。

好比，建一栋大楼，需要设计每一层楼的布局，都有哪些房间、大小怎样，到具体每一间房的格局，什么地方设置门、在什么位置安窗户等。

一般来说，无交互的原型图，属于结构图的范畴。在工作中，为了提高效率，很少画交互型的原型，除非像大厂有专门的交互设计师。

画原型，除了要准确把握需求，还涉及一些人机交互、视觉设计的知识。这里不具体展开。案例中的话费充值功能，简单绘制原型参考如下。

![话费充值页面原型图](/images/0011.jpeg)

## 从动态视角看需求

**从动态的视角去分析需求，则用动态视图，来描述产品的行为性特征，这些特征决定了产品是怎样运行的。**

在 UML 中，动态视图包括活动图、时序图、协作图、状态图。协作图，在产品层面较少使用。活动图，就是常用的流程图。

结合实际工作，总结产品的动态视图有：**流程图、时序图、状态图**。

### 流程图

流程图，是描述为完成某个目标，需要以什么顺序做哪些动作；能直观描述实现目标过程的具体步骤。在很多领域，被广泛使用。

梳理、绘制流程图的过程，也是一种流程化的思考。

UML 中，流程图，也叫活动图；另外，还有泳道活动图。产品上，都经常使用。

#### 普通流程图

普通的流程图，就是在描述产品的具体功能，在具体场景下，是怎么一步步实现的运转过程。

普通流程图中，包括了多个不同对象执行的动作事件，**只能大致描述过程；无法将整个过程中，参与的各个对象体现出来**。

案例需求中，从用户感受到的充值过程，用一般的流程图来梳理，可绘制如下。

![话费充值流程图](/images/0012.jpeg)

#### 泳道活动图

泳道活动图，用来梳理、描述有多个对象参与的流程，对象可以是人，也可以是系统。

泳道活动图，在活动图的基础上，引入了泳道，像游泳比赛的运动员只能在其泳道中比赛一样，规定每个对象的动作只能画在其对应区域。

这样，可以很好地体现整个过程参与的多个对象所做的动作和顺序。

同样是用户充值话费的过程，在产品层面，至少有用户、APP、管理后台、话费供应商（这跟每个公司的业务和系统情况有关，但基本逻辑类似。）的参与才完成。

因此，用泳道活动图来描述，最合适不过。为了避免示例图过于复杂，此处仅画出正常流程，并未包括异常分支。

![话费充值泳道活动图](/images/0023.png)

#### 时序图

时序图，**用于描述产品为实现某一具体目标，多个参与对象之间按时间顺序交互的过程**。

时序图，与泳道活动图类似，不同的是，**时序图更强调对象在交互过程中消息事件的发生顺序**00。

有时为了了解系统性能，或优化体验，要统计某些交互的时长，用时序图，就很方便定义和描述。

用时序图来梳理多个系统间的交互过程，特别好用，我最常使用。时序图画得好，泳道活动图不画都没关系。

同样用户充值话费过程，用时序图来梳理，可以对比下与泳道活动图的区别。

![话费充值时序图](/images/0024.png)

#### 状态图

状态图，**用于描述产品为完成某个目标，某个对象的状态变化和流转过程**。状态，是对象执行或等待某个事件的条件。

常见的，有电商的订单状态、快递物流状态、支付状态等。

系统中对象的状态细化和明确，对监控系统的处理过程，和事后问题排查有很大帮助。

状态图非常重要，又很容易被忽略。以前填过很多坑，就是产品没定义好状态，结果开发按自己想象补上了，事后发现问题，处理起来很麻烦。

如案例中，如今的话费充值，虽然到账时间很快，但订单在系统的流转过程，也有各种状态的变化。

下面以此为例，看看一个比较完整的状态图，可以注意下其与流程图的区别。

![话费充值订单状态图](/images/0025.png)

## 原则与工具

### 基本原则

画图，虽说没有标准答案，但毕竟，产出的可视化图形是后续工作的重要依据，也要给各环节的人阅读使用。

所以，为了保证产出质量和工作效率，还是要满足以下基本原则：

- 逻辑合理、清晰
- 没有疏漏
- 可读性强
- 美观

### 善用工具

画图、分析过程，可用的工具很多，只要功能满足我们的需要，用起来顺手即可。

另外，一定要善于利用工具，让工具为我们所用，可找准一两个工具，用好它，能大大提高效率。

比如，Axure 就非常强大，不仅画原型图，大部分图都可用它完成。

甚至，还能用来写需求文档，这样输出的文档相对统一，便于管理和阅读。

其他常用的，有 Visio ，及专门画思维导图的 MindManager、XMind 等。

其实，只要掌握了方法，哪怕是用纸和笔，照样能描述清楚。

## 回顾总结

总结一下，做产品为什么要画这些图？

**首先，得明白，为什么要画图？**

- 因为，画图，是需求分析的重要环节，是用可视化的方式，对需求进行梳理和展示。
- 能帮助我们梳理、分析需求，更好地理解和分析清楚需求
- 能产出可视化的需求描述，便于阅读使用。

**其次，得知道，为什么画这些图？**

- 因为，画图，有一定的指导思想和方法，能提高准确性和效率。

UML 提供了很好的思想和方法，可以从产品的静态和动态两个视角进行描述。

由此，有了静态和动态两种视图，每个视图有对应的图形，供不同情况使用。

**静态视图，描述产品的结构，决定产品是由什么组成、能做什么、长什么样的，包括：**

- 结构图：功能结构图、信息结构图、产品结构图
- 用例图：业务用例图、系统用例图
- 原型图（无交互）

**动态视图，描述产品的行为，决定了产品是怎样运行的，包括：**

- 流程图：普通流程图、泳道活动图
- 时序图
- 状态图
