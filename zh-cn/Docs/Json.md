# 9 Json
## 9.1 概述
JSON是JavaScript对象标记法，是一种通常用于数据序列化的纯文本数据格式，在RFC 4627标准中定义。JSON格式能够完整地支持 Haystack 类型系统。

## 9.2 类型映射
下列为Haystack和JSON的数据类型映射关系：
```
Haystack      JSON
--------      ----
Grid          Object (specified below)
List          Array
Dict          Object
null          null
Bool          Boolean
Marker        "m:"
Remove        "-:"
NA            "z:"
Number        "n:<float> [unit]" "n:45.5" "n:73.2 °F" "n:-INF"
Ref           "r:<id> [dis]"  "r:abc-123" "r:abc-123 RTU #3"
Str           "hello" "s:hello"
Date          "d:2014-01-03"
Time          "h:23:59"
DateTime      "t:2015-06-08T15:47:41-04:00 New_York"
Uri           "u:http://project-haystack.org/"
Coord         "c:<lat>,<lng>" "c:37.545,-77.449"
XStr          "x:Type:value"
```

注意:
+ 为了更方便解析，进行数字编码时，在浮点值和单位之间使用空格分隔（Zinc中没有空格）
+ 特殊数字使用与Zinc相同的值，如 "INF", "-INF", "NaN"
+ Ref 字符串使用第一个空格将id与字符串的dis部分分开
+ DateTime，Date和Time使用完全符合Zinc所指定的ISO 8601格式
+ DateTime需要有时区名称
+ 包含冒号的字符串必须用 "s:" 前缀编码
+ 没有冒号的字符串可以选择省略 "s:" 前缀
+ 任何没有冒号作为第二个字符的JSON字符串会被当做字符串值

以下为示例：
```
// Haystack
dis: "Site-A", site, area: 5000ft², built: 1992-01-23

// JSON
{"dis":"Site-A", "site":"m:", "area":"n:5000 ft²", "built":"d:1992-01-23"}
```

Haystack和JSON模型非常相似，因为它们都支持相同的核心列表和对象/字典（object/dict）类型。 不同之处在于，Haystack拥有更丰富的标量类型，例如Date，Time，Uri，这些JSON都不能直接支持; 所以我们使用特殊类型的代码前缀将它们编码为字符串。

## 9.3 Grid格式
除了上面定义的灵活类型映射之外，我们还有一个Grid到JSON的映射标准，供REST API使用。

Grid到JSON的映射描述如下：

+ Grid被映射到具有三个字段的JSON对象：元（meta）、列（cols）、行（rows）
+ 元（meta）字段是一个必须含有 "ver" 字段的JSON对象
+ 列（cols）字段是一个列对象的JSON列表
+ 每个列（cols）对象定义一个 "name" 字段和列元数据（column metadata）
+ 行（rows）字段是一个JSON对象的列表
+ 元（Meta）数据和行（row）数据被映射到JSON对象
+ 字典（Dict）值的映射来自于上述类型映射

示例：

```
// Zinc
ver:"3.0" projName:"test"
dis dis:"Equip Name",equip,siteRef,installed
"RTU-1",M,@153c-699a "HQ",2005-06-01
"RTU-2",M,@153c-699a "HQ",1999-07-12

// JSON
{
  "meta": {"ver":"3.0", "projName":"test"},
  "cols":[
    {"name":"dis", "dis":"Equip Name"},
    {"name":"equip"},
    {"name":"siteRef"},
    {"name":"installed"}
  ],
  "rows":[
    {"dis":"RTU-1", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:2005-06-01"},
    {"dis":"RTU-2", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:999-07-12"}
  ]
}
```

这是另一个嵌套列表、字典和网格的例子：

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


// JSON
{
  "meta": {"ver":"2.0"},
  "cols":[
    {"name":"type"},
    {"name":"val"}
  ],
  "rows":[
    {"type":"list", "val":["n:1", "n:2", "n:3"]},
    {"type":"dict", "val":{"dis":"Dict!", "foo":"m:"}},
    {"type":"grid", "val":{
      "meta": {"ver":"2.0"},
      "cols":[
        {"name":"b"},
        {"name":"a"}
      ],
      "rows":[
        {"b":"n:20", "a":"n:10"}
      ]
    }},
    {"type":"scalar", "val":"simple string"}
  ]
}
```