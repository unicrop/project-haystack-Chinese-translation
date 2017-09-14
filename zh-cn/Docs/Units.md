# 5 单位
## 5.1 概述
所有数字标签值都可以用一个可单位进行标注。此外，需要使用 unit 标签对每个数字点进行标注。在这两种情况下，单位必须是由标准单位数据库定义的标识符。

## 5.2 单位系统
作为一般原则，与给定站点相关联的所有数据应仅使用SI（国际单位制）度量系统或美国习惯系统（美式英制单位）。在同一个站点内混合使用不同的度量系统会引发很多麻烦。

## 5.3 数据库
Project Haystack 使用的单位数据由 Fantom 开源社区管理，作为sys::Unit API的一部分。 该数据库最初基于oBIX规范，但是已经被扩展到允许每个单位使用多个别名。

每个测量单位都有全名，以及零个或多个符号，这些符号作为该单位的别名。例如，“square_meter”是全名，符号别名是“m²”。某些单位可能有多个符号，例如“hour”有符号“hr”和“h”。有些单位如“day”没有符号。

所有单位标识符仅限于以下字符：

+ 所有编号超过128的Unicode字符
+ ASCII 字母 a - z 和 A - Z
+ 下划线 _
+ 分隔符 /
+ 百分号 %
+ 美元符号 $

按照惯例，符号是首选的单位表示方法。如果有多个符号，则将单位数据库中定义的最后一个作为首选符号。

## 5.4 Common Units
Below are some commonly used units. You can download the full unit database from Downloads or from the Fantom website.

### Misc
percent, %

### Area
square_meter, m²
square_foot, ft²

### Currency
australian_dollar, AUD
british_pound, GBP, £
canadian_dollar, CAD
chinese_yuan, CNY, 元
euro, EUR, €
us_dollar, USD, $

### Energy
kilowatt_hour, kWh

### Power
kilowatt, kW

### Pressure
kilopascal, kPa
pounds_per_square_inch, psi
inches_of_water, inH₂O
inches_of_mercury, inHg

### Temperature
fahrenheit, °F
celsius, °C

### Temperature differential
fahrenheit_degrees, Δ°F
celsius_degrees, Δ°C

### Time
millisecond, ms
second, sec
minute, min
hour, hr, h
day
week, wk
julian_month, mo
year, yr

### Volumetric Flow
liters_per_second, L/s
cubic_feet_per_minute, cfm

