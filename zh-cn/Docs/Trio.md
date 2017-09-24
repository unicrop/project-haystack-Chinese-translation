# 10 Trio
## 10.1 概述
Trio 代表“文本记录输入/输出”。Trio是一种简单的纯文本格式，用于手写记录定义和其他Haystack标签数据。 它是在 Project Haystack  站点本身创作标签定义的主要格式。

## 10.2 格式
Trio使用简单的纯文本格式设计，便于手动编辑：

+ entities are separated by lines beginning with "-", the lines can have as many dashes as you want
+ each entity is defined by one or more tags
+ one line is used per tag formatted as "name:val"
+ if no value is specified, the value is assumed to be Marker
+ the value is encoded using the same grammar as Zinc
+ string values may be left unquoted if they begin with a non-ASCII Unicode character or contain only the "safe" chars: A-Z, a-z, underbar, dash, or space
+ if a newline follows the colon, then the value is an indented multi-line string terminated by the first non-indented line
+ nested grids are encoded as a multi-line string prefixed with the string value "Zinc:" on the tag line
+ can use // as line comment

Here is a simple example:
```
dis: "Site 1"
site
area: 3702ft²
geoAddr: "100 Main St, Richmond, VA"
geoCoord: C(37.5458,-77.4491)
strTag: OK if unquoted if only safe chars
summary:
  This is a string value which spans multiple
  lines with one or more space characgter
---
name: "Site 2"
site
summary:
  Entities are separated by one more dashes
```

Here is an example with a nested list, dict, and grid value:
```
// Zinc
ver:"3.0"
type,val
"list",[1,2,3]
"dict",{dis:"Dict!" foo}
"grid",<<
  ver:"2.0"
  a,b
  1,2
  3,4
  >>
"scalar","simple string"

// Trio
type:list
val:[1,2,3]
---
type:dict
val:{ dis:"Dict!" foo}
---
type:grid
val:Zinc:
  ver:"3.0"
  b,a
  20,10
```