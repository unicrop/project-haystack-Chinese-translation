# 10 Trio
## 10.1 概述
Trio 代表“文本记录输入/输出”。Trio是一种简单的纯文本格式，用于手写记录定义和其他Haystack标签数据。 它是在 Project Haystack  站点本身创作标签定义的主要格式。

## 10.2 格式
Trio使用简单的纯文本格式设计，便于手动编辑：
+ 实体以“-”开头的行分隔，这些行可以根据需要包含任意多个破折号
+ 每个实体由一个或多个标签定义
+ 每个标签使用一行，其格式为“name:val”
+ 如果没有指定值，则假定该值为Marker
+ 该值使用与Zinc相同的语法进行编码
+ 如果字符串值以非ASCII Unicode字符开头，或仅包含“安全”字符如：A-Z，a-z，下划线，破折号或空格，则可能不引用字符串值
+ 如果换行符在冒号后面，则该值是多行缩进的字符串，并由第一个非缩进的行终止
+ 嵌套网格被编码为标签行上以“Zinc:”字符串值开头的多行字符串
+ 可以“//”用作行注释

这是一个简单的例子：
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
这是一个嵌套列表，dict和grid值的示例：
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