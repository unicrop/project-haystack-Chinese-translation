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

You use the -> to dereference a tag which has a Ref value. For example if your equip rec has a siteRef tag that references the site, you can query for equip in a given city such as:
```
equip and siteRef->geoCity == "Chicago"
```

The way to read the above expression is match an entity if:

+ it has equip tag
+ and it has a siteRef tag which is a Ref
+ and what the siteRef tag points to has the geoCity tag
+ and that the site's geoCity tag is equal to "Chicago"

## 7.3 Grammar
The formal grammar of the filter langauge:
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

See [Zinc grammar]() for productions reused from Zinc. Note that Marker, Bin, and DateTime scalars are not supported. Bools are encoded as "true" or "false" (Zinc encodes as "T" or "F").