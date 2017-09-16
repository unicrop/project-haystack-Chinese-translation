# 6 网格（Grids）
## 6.1 概述
网格是标签化实体的二维表格形式。我们使用网格作为核心数据模型，并在 HTTP 层面使用[Rest]() API来序列化被 haystack 标记的数据。

## 6.2 结构
一个网格由以下部分组成：

+ 网格级元数据
+ 一个或多个列，列包含可程序化的名称和元数据
+ 零个或多个行，行包含多个标量化的单元格

元数据只是由[标签模型]()指定的键值对标签的列表。网格级元数据允许我们指定关于整个网格的标签。

列和行用于建立 Haystack 标签实体列表。列通过实体列表中所有唯一标签名的并集计算得出。每列由编程名称组成，该名称必须是有效的[标签名称]()。列也可以使用标签指定元数据。[dis]() 列标签可以用于为列提供人性化的显示名称（因为编程名称可能是驼峰式命名方式）。

行用于对标签实体的列表进行建模。行由网格中每列的单元格组成。每个单元格必须是有效的[标签种类]()标量值，如果该行没有给定列的值，则可能为 null 或 空白。

## 6.3 范例
下面是三个站点模型的 haystack 实体：
```
id: @site-a
dis: "Site A"
site
area: 45000ft²

id: @site-b
dis: "Site B"
site

id: @site-c
dis: "Site C"
site
area: 62000ft²
phone: "(804) 555-1234"
```

上面的三个实体都有一个id，dis和site标签。另外两个有 area 标签，最后一个有 phone 标签。要将这三个实体组合成一个网格，我们最终得到五列和三行：
```
id       dis       site  area      phone
-------  -------   ----  --------  ------
@site-a  "Site A"  ✓     45000ft²
@site-b  "Site B"  ✓
@site-c  "Site C"  ✓     62000ft²  "(804) 555-1234"
```

注意列是由实体贡献的所有标签的并集。因为并不是每个实体共享相同的列，所以我们有空或 null 的单元格。我们可以进一步添加网格级或列级元数据。