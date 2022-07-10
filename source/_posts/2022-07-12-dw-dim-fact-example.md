---
layout: post
title: 数据仓库【十二】：星型模型设计实例
date: 2022-07-12
tags:
- Data Warehouse
- 数据仓库
categories: 数据仓库
description: 星型模式将业务流程分为事实和维度。事实包含业务的度量，是定量的数据，如销售价格、销售数量、距离、速度、重量等是事实。维度是对事实数据属性的描述，如日期、产品、客户、地理位置等是维度。一个含有很多维度表的星型模式有时被称为蜈蚣模式，显然这个名字也是因其形状而得来的。蜈蚣模式的维度表往往只有很少的几个属性，这样可以简化对维度表的维护，但查询数据时会有更多的表连接，严重时会使模型难于使用，因此在设计中应该尽量避免蜈蚣模式。
---

假设有一个连锁店的销售数据仓库，记录销售相关的日期、商店和产品，其星型模式如下所示：

![](/images/0076.png)

Fact_Sales是唯一的事实表，Dim_Date、Dim_Store和Dim_Product是三个维度表。每个维度表的Id字段是它们的主键。事实表的Date_Id、Store_Id、Product_Id三个字段构成了事实表的联合主键，同时这个三个字段也是外键，分别引用对应的三个维度表的主键。Units_Sold是事实表的唯一一个非主键列，代表销售量，是用于计算和分析的度量值。维度表的非主键列表示维度的附加属性。

下面的查询可以回答2015年各个城市的手机销量是多少。

```SQL
select s.city as city, sum(f.units_sold)
    from fact_sales f
    inner join dim_date d on (f.date_id = d.id)
    inner join dim_store s on (f.store_id = s.id)
    inner join dim_product p on (f.product_id = p.id)
    where d.year = 2015 and p.product_category = 'mobile'
    group by s.city;
```
