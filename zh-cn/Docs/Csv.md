# 11 Csv
## 11.1 概述
CSV代表逗号分隔值。它是通常用于表格数据序列化的纯文本格式。它在[RFC 4180]()中规定。 CSV提供了一种将表格数据置入其他应用程序的简单方法，如Excel。

## 11.2 网格格式
Haystack定义了供[REST API]()使用的网格到CSV的标准映射。由于它很简单，CSV不能为Haystack网格模型提供完整的保真度 -- 元数据和类型信息被丢弃。

网格到CSV的映射如下：
+ 第一行是列的显示名称（编程名称和元数据被丢弃）
+ 随后的行映射到网格中的行
+ 标记单元格被编码为Unicode的 ✓ 号，其Unicode码为U+2713
+ Null 单元格被编码为空字符串
+ Ref 单元格被编码为“@id dis”, 例如“@3ef7 Site-1”
+ Bools 被编码为“true”或“false”
+ Strs 和 Uris 使用它们的未转义值进行编码
+ 其他的都使用其[Zinc]()格式进行编码

例子：
```
// Zinc
ver:"3.0" projName:"test"
dis "Equip Name",equip,siteRef,installed
"RTU-1",M,@153c600e-699a1886 "HQ",2005-06-01
"RTU-2",M,@153c600e-699a1886 "HQ",1999-07-12

// CSV
Equip Name,equip,siteRef,installed
RTU-1,✓,@153c600e-699a1886 HQ,2005-06-01
RTU-2,✓,@153c600e-699a1886 HQ,1999-07-12
```
