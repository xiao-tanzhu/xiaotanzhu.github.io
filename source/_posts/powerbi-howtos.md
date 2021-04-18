---
layout: post
title: "PowerBI常见操作"
date: 2021-04-18
tags: ["PowerBI", "BI"]
categories: [PowerBI]
description: "常见的一些PowerBI的使用方法"
---

## 分析方法

### 按周进行数据分析

Source：https://zhuanlan.zhihu.com/p/58948031

## 类Excel的计算

### VLOOKUP (DAX)

Document: https://docs.microsoft.com/en-us/dax/lookupvalue-function-dax
Example: https://www.wallstreetmojo.com/vlookup-in-power-bi/

#### Syntax

```
LOOKUPVALUE(
    <result_columnName>,
    <search_columnName>,
    <search_value>
    [, <search2_columnName>, <search2_value>]…
    [, <alternateResult>]
)
```

#### Parameters

|Term|	Definition|
|----|------------|
|result_columnName	|The name of an existing column that contains the value you want to return. It cannot be an expression.|
|search_columnName	|The name of an existing column. It can be in the same table as result_columnName or in a related table. It cannot be an expression.|
|search_value	|The value to search for in search_columnName.|
|alternateResult	|(Optional) The value returned when the context for result_columnName has been filtered down to zero or more than one distinct value. When not provided, the function returns BLANK when result_columnName is filtered down to zero value or an error when more than one distinct value.|

#### Return value

The value of result_column at the row where all pairs of search_column and search_value have an exact match.

If there's no match that satisfies all the search values, BLANK or alternateResult (if supplied) is returned. In other words, the function won't return a lookup value if only some of the criteria match.

If multiple rows match the search values and in all cases result_column values are identical, then that value is returned. However, if result_column returns different values, an error or alternateResult (if supplied) is returned.
