# 6 网格（Grids）
## 6.1 概述
网格是标签化实体的二维表格形式。我们使用网格作为核心数据模型，并在 HTTP 层面使用[Rest]() API来序列化被 haystack 标记的数据。

## 6.2 结构
一个网格由以下部分组成：

+ 网格级元数据
+ 一个或多个列，列包含可程序化的名称和元数据
+ 零个或多个行，行包含多个标量化的单元格

元数据只是由[标签模型]()指定的键值对标签的列表。网格级元数据允许我们指定关于整个网格的标签。

Columns and rows are used to model a list of Haystack tagged entities. Columns are computed by the union of all unique tag names in a list of entities. Each column is composed of a programmatic name which must be a valid [tag name](). Columns may also specify metadata using tags. The [dis]() column tag may be used to provide a human friendly display name for the column (since the programmatic name will likely be camel case).

Rows are used to model the list of tagged entities. Rows are composed of a cell for each column in the grid. Each cell must be a valid [tag kind]() scalar value or may be null/sparse if the row doesn't have a value for a given column.

## 6.3 Example
Consider three haystack entities which model sites:
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

Our three entities all have an id, dis, and site tags. In addition two have the area tag, and one has a phone tag. To combine these three entities into a grid we end up with five columns and three rows:
```
id       dis       site  area      phone
-------  -------   ----  --------  ------
@site-a  "Site A"  ✓     45000ft²
@site-b  "Site B"  ✓
@site-c  "Site C"  ✓     62000ft²  "(804) 555-1234"
```

Note the columns are union of all tags shared by the entities. Because not every entity shares the same columns, we have sparse or null cells. We could further add grid level or column level meta.