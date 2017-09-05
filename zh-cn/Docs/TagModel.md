# 2 标签模型
## 2.1 元模型
在Project Haystack中，我们定义了建筑设备和运营的模型。但就像每个模型一样，我们必须为我们的模型使用一些框架。或者换句话说，什么是模型的模型-元模型？

如果我们以Java编程语言构建模型，则元模型将是Java类，我们的分类法将被定义为面向对象的类层次结构。如果使用关系数据库，关系模式将是我们的元模型。 在oBIX中，元模型是oBIX协议。在RDF中，元模型是主语-谓语-宾语 - 对象三元组。

我们的操作数据的原始来源很可能已经在上面定义的元模型之中。 然而，像面向对象的类层次结构或关系型数据库这样的刻板结构并不适合楼宇自动化系统的领域，因为每个项目基本上都是独一无二的。 考虑一个AHU（空气处理机），是否有一类层次结构可以描述AHU功能的每个独特组合？ 由于AHU往往是针对每个设施定制的，因此任何一个单一的固定架构将不可能在多个项目中使用。

为了解决这些独特的挑战，Project Haystack使用了一种非常简单的元模型：标签。标签是可以与像AHU这样的实体相关联的键/值对。 因为标签是简单和动态的，它们可以非常灵活地构建标准化模型，这些模型可以在每个项目或每个设备的基础上轻松定制。 此外，标签模型可以轻松地集成或分层在传统模型（例如面向对象类或关系模式）之上。

## 2.2 实体
实体是现实世界中一些物理对象的抽象。 实体包括站点，设备，传感器点，气象站等。在软件系统中，实体可能被建模为数据库中的记录，楼宇自动化系统中的对象，或者也可能只是CSV文件中的一行。

Haystack没有对实体的存储或管理方式进行任何具体设计，而是定义如何使用特定键/值对来标记这些实体。 通过使用标准化标签库，我们可以构建一个允许行业内语义理解的分类法。最终的结果是，我们都可以省钱地将数据从一个专有系统映射到另一个专有系统。

如果一个实体的具体定义被存储在数据库中，那么我们将使用术语record或rec与实体（entity）互换。我们经常使用REST API中的术语rec来描述客户端与haystack实体的服务器数据库间的交互。

## 2.3 标签
标签是应用于实体的键/值对。标签定义关于实体的事实或属性。例如，如果我们将 site 标签应用于某个实体，意味着我们声明该实体代表一个建筑物。 如果我们还添加了 geoAddr 标签，我们则声明了此建筑物的街道地址。

### 2.3.1 标签名称
标签名称仅限于以下字符：

+ 必须以ASCII码小写字母(a-z)开头
+ 必须只包含ASCII码字母，数字或下划线(a-z, A-Z, 0-9, _)
+ 按惯例使用驼峰命名法 (fooBarBaz)

限制使用标签名称，确保它们可以方便地用作编程语言和数据库中的标识符。
Restricting tag names, ensures they may be easily used as identifiers in programming languages and databases.

### 2.3.2 标签种类
一个标签种类是指标签所允许某个值类型。以下是原子标量标签类型：
+ **Marker:** 标签只是一个标记注释，不包含有意义的值。标记标签用于表示“类型”或“is-a”关系。
+ **Bool:** 布尔类型表示“true”或“false”。
+ **NA:** 单例值，表示缺失数据，且无法获取。
+ **Number:** 某点位的整型或浮点型数据，并标注可选的测量单位。
+ **Str:** Unicode字符串。
+ **Uri:** 统一资源标识符。
+ **Ref:** 另一个实体的引用。Haystack没有规定具体的身份或引用机制，但它们应该是交叉链接实体的一种方式。另请参阅[Containment]()。我们以“@”字符开头格式化 Ref，并且需要使用特定的ASCII字符子集：a-z，A-Z，0-9，下划线，冒号，破折号，点或波浪号。
+ **Bin:** 格式为Bin(text/plain)的MIME类型的二进制大对象。
+ **Date:** ISO 8601标准日期，例如“年月日”表示为：2011-06-07。
+ **Time:** ISO 8601标准时间，例如“时分秒”表示为：09:51:27.354。
+ **DateTime:** ISO 8601标准时间戳，后跟时区名称：
```
2011-06-07T09:51:27-04:00 New_York
2012-09-29T14:56:18.277Z UTC
```
+ **Coord:** 纬度/经度的地理坐标格式为C(lat,lng)。
+ **XStr:** 扩展类型的字符串，其指定一个类型名称和一个字符串编码。

有三种集合标签类型：

+ **List:** 零个或多个值的列表
+ **Dict:** 相关联的键/值标签对数组
+ **Grid:** 包含行和列的二维表，请参阅[Grids]()

## 2.4 Id
The id tag is used model the unique identifier of an entity in system using a Ref value type. The scope of an entity is undefined, but must be unique with a given system or project. This identifier may be used by other entities to cross-reference using tags such as siteRef, ahuRef, etc.

## 2.5 Dis
The dis tag is used with all entities as the standard way to define the display text used to describe the entity. Dis values should be short (less than 30 or 40 characters), but fully descriptive of the entity.

## 2.6 Example
Let's look a simple example for an entity describing a site:
```
id: @whitehouse
dis: "White House"
site
area: 55000ft²
geoAddr: "1600 Pennsylvania Avenue NW,  Washington, DC"
tz: "New_York"
weatherRef: @weather.washington
```
In the example above we have an entity with seven tags: id, site, dis, area, geoAddr, tz, and weatherRef. By convention when writing examples we will list each tag on their own line or separated by a comma. The site tag has no explicit value, so it is assumed to be marker tag. The dis, geoAddr, and tz tags have string values indicated by double quotes. The area tag has a number value indicated by a scalar with unit of square feet. The weatherRef tag is a reference to another entity, which we indicate using the "@" character.



