---
layout: post
title: 一文读懂电商产品架构
date: 2022-02-17
tags:
- 电商
- 产品经理
- 产品架构
categories: 产品经理
description: 刚接触电商领域的你是否有很多疑惑？对于许多电商产品是否感到无从下手？这篇文章作者详细介绍了电商产品的构架，推荐想要了解电商产品构架的童鞋阅读。
---

大部分刚接触电商领域的产品经理，或多或少会体会到电商系统的模块多、流程长、逻辑复杂。电商系统中，除了前端APP和网站看得见摸得着外，更多的是看不见的底层逻辑和后端系统。

对于大部分同学而言，要找到同类后端系统的竞品进行研究借鉴是一件比较麻烦的事儿，因此长期以来便形成了一种电商产品的门槛比其他行业的产品门槛要更高的错觉。

虽然电商产品比较复杂，但是它有一个很重要的特征，那就是**电商产品属于典型的业务驱动型产品**。业务驱动型产品，其所有功能都服务于一个实体业务。因此想要弄清楚电商产品有哪些功能模块，可以从电商的业务入手，先了解一个完整的电商业务有哪些环节，基于每个环节再分析需要哪些功能来支撑，将所有功能穷举完以后，再按照一定的逻辑进行归纳总结即可窥探电商产品的全貌了。

本文将从电商的**业务流程**到电商的**系统流程**再到电商的**系统架构**这三个部分展开，分别通过三张图来深入浅出展示电商的世界。

## 电商核心业务-采销仓配

电商发展至今，出现了各种各样的叫法，有平台电商、自营电商、B2B、B2C、生鲜电商、社交电商、兴趣电商等等。不论电商的形式如何多样，其本质都是买卖双方围绕商品交易履约的过程。

在开始探讨之前，我们先圈定一下探讨的方向，**本文主要以自营电商为探讨方向**。因为与之相对的平台型电商的核心其实是流量生意，系统的责任更多的是将海量的商品和消费者做匹配，平台以此向商家收取交易佣金和广告费。

而自营电商除了需要具备平台电商的能力之外，由于自采自销的业务模式还涉及采购、仓储、履约等实体环节，其系统模块更多更长更复杂，基本上能覆盖平台型电商的核心系统模块，因此以自营电商为主要探讨方向可以了解的更为全面。而其他诸如B2B、B2C、生鲜电商等只是领域或表现形式不同，不属于同一个维度。

那么一个完整的自营电商业务是怎么样的？将一件商品交付给消费者需要经历哪些环节呢。

![电商业务流程](/images/0017.jpeg)

通过以上业务场景图可以看出，一件商品卖给消费者需要经历的环节非常多，包括线下实体环节和线上系统环节，按照线下实体环节进一步抽象后，可以将自营电商业务划分为以下4个部分，分别是。

1. 从供应商处采购产品
1. 采购产品入仓存储管理
1. 商品上架到电商平台销售
1. 根据销售订单进行履约配送

用业务流程图表示如下。

![电商业务流程](/images/0018.jpeg)

了解完业务流程后，基于业务流程再来梳理这些业务环节背后所需的系统就比较容易了。电商业务中所有的系统、功能都是基于以上业务流程所产生和演化的。

## 电商·系统全流程

基于上面的业务梳理结果，我们将业务流程和所需系统或功能结合起来，看看每个业务环节都有哪些系统或功能，同时，我们将整个系统流程按采销仓配的业务边界进行划分，再结合C端购买流程，就能得到一张完整的大型电商系统流程图，如下。

![电商业务流程](/images/0001.jpg)

上图就是一个较为复杂的电商业务在系统中的流转逻辑（参考京东），从供应商/卖家入驻平台到采购商品到收货入库到上架销售再到配送履约，通过这张图就可以比较清晰的看出每个业务环节所对应的系统或功能大概有哪些了。为了便于初学者理解，有必要对流程图中的的部分节点进行解释说明（可结合流程图图对比阅读）。

### 入驻&采购流程

1. **入驻：**包括供应商入驻和卖家入驻，入驻动作主要包含选择入驻企业类型、基于类型提交相应材料（营业执照、资质等），签订合同，经过平台审核通过后，再开通企业账户钱包。
1. **开店：**对于卖家而言，入驻成功后即可创建店铺，创建店铺主要是根据营业执照的经营范围选择主营类目、填写店铺基本信息、缴纳质保金等。
1. 采购供应商入驻后，自营采销在采购系统中下**采购订单**，订单通过EDI或者线下的方式推送给供应商。
1. **供应商发货：**供应商收到采购订单后，根据采购单中的信息（商品、收货仓库等）发货，发货后即产生采购在途库存。
1. **库存：**无论是在途库存，还是实物入库后产生的实物库存，以及前端的可售库存等，均由库存中心控制。

### 销售流程

1. **发布商品。**即创建商品sku，包括填写商品参数，商品详情介绍等信息，POP卖家会直接设置价格，自营因品类而异，创建商品时会调用主数据，即spu。
1. **定价。**发布商品时会通过定价系统制定销售价格，销售价格与采购价、库存成本价、促销价等一系列价格组成复杂的价格模型，并一起记录在价格中心，形成完善的价格体系。
1. **上架。**发品并定价完成后，可通过上下架功能控制商品上架销售。
1. **促销。**即通过营销工具做促销活动，包括单品类、总价类、优惠券等，所有的促销规则均通过营销中心控制，包括活动准入规则、不同活动之间的叠加互斥规则、促销命中规则、优惠计算、促销风控等。
1. **活动页搭建。**促销商品有时候会在特定的活动专题页面集中展示（满减专区、特价专区等），通过CMS系统可以随时随地的通过拖拽组件的方式搭建一个任意样式的专题页面，而不用临时开发。

### 黄金流程

指在用户视角下购买商品时所经历的页面，包括**搜索—列表—商品详情页—购物车—提单页**，很多公司内部也叫交易主流程或交易动线；

黄金流程是价格和促销的主要体现环节，因此在在黄金流程中，对于价格和促销的设计尤为重要，包括优惠价格的高亮突出，促销标识的设计，购物车的凑单分组等，对转化有直接的影响。

1. **生成订单：**生成订单在后端是一个非常复杂的过程，包括校验库存、价格、库存、用户信息、促销信息等，订单由订单中心负责创建生成。
1. **预占库存：**生成订单的同时，会与库存中心交互预占库存。
1. **收银台：**订单生成和支付是两个独立的环节，常见的支付方式有【在线支付】和【货到付款】，部分业务（B2B）会设置【对公转账】支付方式，后面两种支付方式以线下的形式完成。对于在线支付的订单，则可以将多种支付方式聚合到收银台页面，包括自由支付、网银支付和三方支付（微信支付、支付宝支付、京东支付等）。
1. **支付对账。**对于先款后货订单，各类支付方式均设有支付额度限制，对公转账的方式也无法保证用户一定按照订单实收支付，因此存在实际支付金额不一定等于订单应收金额，所以需要经过对账系统完成对账。

### 订单寻源履约中心

介于交易流程和仓储流程之间的系统——将电商系统生成的数以万计的订单，按时下发给最适合的库房进行生产，就是订单寻源履约系统的职责。订单寻源履约中心由多个子系统或服务组成，包括拆分系统、分摊计算服务、转移系统、履约控制中心等。

1. **订单拆分：**对账成功后，订单会进入到寻源履约中心的第一个系统——拆分系统。拆分的场景主要有：生产维度（仓库不同、商家不同、配送方式不同···）、业务类型维度（实物订单、虚拟订单、生鲜订单···）等，该系统的职责是按照不同的规则将父订单拆分成多个子订单进行生产。拆分时会调用分摊计算服务计算每个新子订单的金额。
1. **订单转移。**订单拆分完成之后会流转到转移流程，订单拆分完成之后会生成不同类型的订单，经过转移之后，不同的订单要执行不同的生产时机、不同的生产地点、不同的生产流程。
    1. 针对**虚拟订单**，不用经由库房实际生产，直接由转移给订单中心或者虚拟业务对应的系统进行处理。
    1. 针对**厂商直发订单和POP订单**，因为这类订单都是由三方卖家发货或者工厂直接发货，转移系统会与对应的业务系统交互，生成供应商采购单或下传订单给POP对应的系统。
    1. 针对**自营的订单**，由于订单商品所属的仓库不同、配送时效不同、商品的类型不同（如生鲜如要走冷链库房生产）、用户指定送达时间等原因，转移系统会控制订单生产的时机（如一周后送达订单则需要控制不要立刻下传库房）、生产的流程，生产的仓库。
1. **履约控制：**履约控制系统负责控制订单生产的流程，将订单推送至对应的库房，并回传生产节点（拣货、复核、打包出库···）给前台系统；针对取消逆向订单，根据订单流转的不同节点做对应的控制：如订单未流转到仓库，则负责暂停订单下传，订单未出库则负责通知仓库终止生产、订单未派件则通知配送系统终止派件等。
1. **订单下传。**订单经过拆分、转移、履约控制后，按时下传到对应的库房进行生产。

以上就是电商全流程的介绍，全流程中的每个流程节点都是一个独立的、庞大的系统，本文只是介绍各个系统之间的流转关系和基本职责，详细的功能设计将在后续的系列文章中展开。

## 大型电商产品架构

我们通过电商业务流程得到了系统流程，有了系统流程就得到了电商系统的基本功能模块，我们基于上面梳理出来的基础功能模块，再从系统全局的角度进行扩展和做更细粒度的拆分，将最终拆分出的功能模块按照架构图的逻辑进行组织，就得到了一下产品架构图。

![电商产品架构](/images/0002.jpg)

这张架构图的组织逻辑比较简单（可结合架构图对比查看）。

从上到下分别是：**用户端系统>运营系统>履约系统>生产系统>基础平台>BI系统**。

- **用户端系统：**主要负责用户选购商品的需求，核心系统包括注册/登录、黄金流程和个人中心等；用户端系统属于前台系统，在产品设计上更注重用户体验、数据分析等。

- **运营系统：**主要承载了内部运营的能力，核心系统包括用户管理、商品管理、价格管理以及营销管理等。

- **交易履约系统：**交易履约系统是一个中枢系统，向上承载订单交易，向下控制生产履约，核心系统有订单中心和寻源履约中心，属于电商系统中比较黑盒的部分，界面较少，更多的是底层逻辑。

- **供应链与生产系统：**供应链系统即进销存系统，在很多企业中统称ERP，主要负责商品的采购、库存的管理等、仓储管理（WMS）以及运输管理（TMS）。

- **基础平台：**基础平台是业务系统之外的系统，主要包括员工账号管理、主数据、财务系统、商家管理系统、以及服务市场和开放平台等。

- **BI系统：**平行于电商的其他系统，采集各个系统产生的数据，加工处理后反哺其他系统，提供各类数据分析能力。

如果将电商系统比作一个大型超市。

- **用户端系统**就是可以看见的超市本身，首页就是超市的入口，搜索就是导购牌，列表就是货架，购物车就是购物车，提单就是出口收银台，用户端系统负责承载消费者的选购下单需求；
- **运营系统**就是超市的营销导购人员，负责引导消费者购物、负责制定商品价格、发布促销活动，CMS系统就是超市的DM单、易拉宝，运营系统向上支撑各渠道、各终端的销售需求，向下对接订单系统和交易履约系统；
- **交易履约系统**就是超市的调度员，对消费者的订单负责，当超市缺货时，及时从附近仓库调货，或者向工厂订货，保证消费者的订单能按时履约；
- **供应链与生产系统**就是超市的幕后工作者，包括采购员、库管、配送员等。

## 小结

1. 一个完整的自营电商业务有4个核心板块：采购、销售、仓储、配送；
2. 基于电商产品均服务实体业务的特性，根据业务流程可以梳理出每个环节对应所需的系统或功能，将这些功能按照业务流程串联起来就能得到完整的系统流程图；
3. 将系统功能拆分成尽可能细的粒度，再按照一定的组织逻辑就能得到最终的架构图。

就像前文说的，上述的每一个模块都是一个独立的、庞大的系统，在一个成熟的电商公司中，每一个系统往往都有一个或多个产品经理专门负责这个系统的设计、迭代和优化。

电商发展至今已经成为一个相对成熟的领域，很多系统和功能都能在网上找到对应的设计方案，可能只是行业、业务体量或者方向上有所差别。建议初学者在设计电商的核心系统时，不要凭借主观感觉盲目设计，应深入调研已有的成熟方案，避免踩坑。