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

## 5.4 常用单位
以下是一些常用的单位。您可以从 [下载链接]() 或从 [Fantom]() 网站下载完整的数据库。

### 杂项
percent, %（百分比）

### 面积
square_meter, m²（平方米）
square_foot, ft²（平方英尺）

### 货币
australian_dollar, AUD（澳元）
british_pound, GBP, £（英镑）
canadian_dollar, CAD（加拿大元）
chinese_yuan, CNY, 元
euro, EUR, €（欧元）
us_dollar, USD, $（美元）

### 能源
kilowatt_hour, kWh（千瓦时）

### 功率
kilowatt, kW（千瓦）

### 压力
kilopascal, kPa（千帕）
pounds_per_square_inch, psi（磅/平方英寸）
inches_of_water, inH₂O（英寸水柱）
inches_of_mercury, inHg（英寸汞柱）

### 温度
fahrenheit, °F（华氏度）
celsius, °C（摄氏度）

### 温差
fahrenheit_degrees, Δ°F（华氏度）
celsius_degrees, Δ°C（摄氏度）

### 时间
millisecond, ms（毫秒）
second, sec（秒）
minute, min（分钟）
hour, hr, h（小时）
day（天）
week, wk（周）
julian_month, mo（朱利安月）
year, yr（年）

### 体积流量
liters_per_second, L/s（升/秒）
cubic_feet_per_minute, cfm（立方英尺每分钟）

