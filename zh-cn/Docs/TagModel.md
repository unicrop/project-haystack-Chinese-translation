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

## 2.3 Tags
A tag is a name/value pair applied to an entity. A tag defines a fact or attribute about an entity. For example if we apply the site tag to an entity, then we are declaring that the entity represents a building. If we also add the geoAddr tag we are declaring the street address of the building.

### 2.3.1 Tag Names
Tag names are restricted to the following characters:

+ Must start with ASCII lower case letter (a-z)
+ Must contain only ASCII letters, digits, or underbar (a-z, A-Z, 0-9, _)
+ By convention use camel case (fooBarBaz)

Restricting tag names, ensures they may be easily used as identifiers in programming languages and databases.

### 2.3.2 Tag Kinds
A kind is one of the permitted value types of a tag. The following are the atomic scalar tag kinds:

+ **Marker:** the tag is merely a marker annotation and has no meaningful value. Marker tags are used to indicate a "type" or "is-a" relationship.
+ **Bool:** boolean "true" or "false".
+ **NA:** singleton value which represents not available for missing data
+ **Number:** integer or floating point number annotated with an optional unit of measurement.
+ **Str:** a string of Unicode characters.
+ **Uri:** a Unversial Resource Identifier.
+ **Ref:** reference to another entity. Haystack doesn't prescribe a specific identity or reference mechanism, but they should be some way to cross link entities. Also see Containment. We format refs with a leading "@" and require a specific subset of ASCII characters be used: a-z, A-Z, 0-9, underbar, colon, dash, dot, or tilde.
+ **Bin:** a binary blob with a MIME type formatted as Bin(text/plain)
+ **Date:** an ISO 8601 date as year, month, day: 2011-06-07.
+ **Time:** an ISO 8601 time as hour, minute, seconds: 09:51:27.354.
+ **DateTime:** an ISO 8601 timestamp followed by timezone name:
```
2011-06-07T09:51:27-04:00 New_York
2012-09-29T14:56:18.277Z UTC
```
+ **Coord:** geographic coordinate in latitude/longitude formatted as C(lat,lng)
+ **XStr:** extended typed string which specifies a type name a string encoding

There are three collection tag kinds:

+ **List:** list of zero or more values
+ **Dict:** an associated array of name/value tag pairs
+ **Grid:** a two dimensional table of columns and rows, see Grids 

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



