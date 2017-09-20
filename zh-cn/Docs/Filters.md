# 7 过滤器
## 7.1 概述
过滤器是用于构造匹配实体的谓词的简单查询语言。Rest[读操作]()使用过滤器来对服务器执行临时查询。

## 7.2 用法
简单的用法只是一个标签名称，它匹配包含标签的任何记录（不管其值）：

```
site  // 查询任何含有 "site" 标签的记录
```
要匹配标签值，您可以使用任何等于或比较运算符：
```
geoPostalCode == "23220"   // 等于
geoPostalCode != "23220"   // 不等于
curVal < 75                // 小于
curVal <= 75               // 小于等于
curVal > 75                // 大于
curVal >= 75               // 大于等于
```
要比较的标量是使用 [Zinc]() 格式进行编码的（除了以下几个例外）。

你可以使用 and，or 或 not进行组合过滤：
```
site or equip             // 包含 site 或 equip 标签
equip and hvac            // 包含 equip 和 hvac 标签
equip and not ahu         // 包含 equip 标签, 但没有 ahu 标签
```

您可以使用 -> 来引用具有Ref值的标签。例如，如果您的 equip 记录有一个引用到站点的 siteRef 标签，则可以查询给定城市中的设备，例如：
```
equip and siteRef->geoCity == "Chicago"
```

读取上述表达式的方式是匹配一个实体，如果：

+ 它有一个 equip 标签
+ 并且它有一个 siteRef 标签，该标签是一个 Ref
+ 并且 siteRef 标签所指的站点含有 geoCity 标签
+ 并且该站点的 geoCity 标签等于 "Chicago"

## 7.3 语法
过滤器语言的正式语法：
```
<filter>     :=  <condOr>
<condOr>     :=  <condAnd> ("or" <condAnd>)*
<condAnd>    :=  <term> ("and" <term>)*
<term>       :=  <parens> | <has> | <missing> | <cmp>
<parens>     :=  "(" <filter> ")"
<has>        :=  <path>
<missing>    :=  "not" <path>
<cmp>        :=  <path> <cmpOp> <val>
<cmpOp>      :=  "=" | "!=" | "<" | "<=" | ">" | ">="
<path>       :=  <name> ("->" <name>)*

<val>        :=  <bool> | <ref> | <str> | <uri> |
                 <number> | <date> | <time>
<bool>       := "true" or "false"
<number>     := same as Zinc (keywords not supported INF, -INF, NaN)
<ref>        := same as Zinc
<str>        := same as Zinc
<uri>        := same as Zinc
<date>       := same as Zinc
<time>       := same as Zinc
```

对于那些复用 Zinc 的软件产品，可以参见[Zinc语法]()。请注意，它不支持Marker，Bin和DateTime标量。Bools被编码为“true”或“false”（Zinc将其编码为“T”或“F”）。

