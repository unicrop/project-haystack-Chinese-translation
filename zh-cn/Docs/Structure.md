# 3 结构
## 3.1 概述
Haystack模型的主要结构是基于三个实体的层次结构：

+ Site: 拥有街道地址的单一建筑
+ Equip: Site 内的物理或逻辑设备
+ Point: 设备的传感器，执行器或设定值

此三级层次结构定义了跨项目使用的主要实体。 其他核心实体包括：

+ Weather: 外部天气条件

下图说明了这个基本的三级层次结构以及它们如何交叉引用：

![](siteEquipPoint.png)

## 3.2 容器(Containment)
Haystack本身不是基于“树结构”，但是可以使用[引用标签](TagModel#tagKinds)定义树结构。 由于给定的实体可以有多个引用标签，因此可以很容易地定义多维树结构。

核心 Site/Equip/Point 模型用作主干树结构和基本框架。然而，替代结构对于分析可能同样重要：

+ Electrical Distribution: 电表，子电表和电气负载如何相关？
+ Air Distribution: AHU，VAV和区域如何相关？
+ Chilled Water/Steam Distribution: AHU，中央设备，锅炉和制冷机如何相关？

由于领域的复杂性，您不应该假设任何一个树结构都可用于完全描述建筑物及其设备。最好把 Haystack project 看作一个数据图，其中实体使用引用标签定义了多种关系。

## 3.3 站点(Site)
站点实体使用 site 标签对单个设施进行建模。在对任何楼宇进行建模时，一个很好的经验法则是将其街道地址作为自己的站点。 例如，在对校园进行建模时，更好的方式是把每个建筑物作为一个站点，而不是把整个校园作为一个站点。

站点使用的核心标签:

+ geoAddr: 自由格式的站点地理地址（可能包括其他地理位置标签，如geoCity或geoCoord）
+ tz: 站点所在的时区
+ area: 设施的平方英尺或平方米。这使得站点通过面积进行规范。
+ weatherRef: 将该站点与气象站相关联，用以可视化天气状况并执行基于天气的能量归一化
+ primaryFunction: 描述建筑物主要功能的枚举字符串
+ yearBuilt: 建筑物四位数的建造年份

以下是一个使用地理位置标签充分展示一个站点实体的示例：
```
id: @whitehouse
dis: "White House"
site
area: 55000ft²
tz: "New_York"
weatherRef: @weather.washington
geoAddr: "1600 Pennsylvania Avenue NW, Washington, DC"
geoStreet: "1600 Pennsylvania Ave NW"
geoCity: "Washington D.C."
geoCountry: "US"
geoPostalCode: "20500"
geoCoord: C(38.898, -77.037)
```

## 3.4 设备
设备使用 equip 标签进行建模。设备通常是一个物理资产，如AHU，锅炉或冷水机。然而，equip标签也可以用于对诸如冷水机组之类的逻辑分组进行建模。

所有设备应使用 siteRef 标签在单个站点内进行关联。反过来，设备通常会包含使用 equipRef 标签与该设备相关联的点。

以下是一个AHU设备实体的示例：
```
id: @whitehouse.ahu3
dis: "White House AHU-3"
equip
siteRef: @whitehouse
ahu
```
equipRef 标签可以根据需要用于设备实体，以此来对嵌套设备和容器关系进行建模。

## 3.5 点
点通常是数字或模拟传感器或执行器实体（有时称为硬点）。点也可以表示一个配置值，例如设定值或计划日志（有时称为软点）。点实体用 point 标签标记。

所有点进一步分类为传感器，指令或设定点，使用以下三个标签之一：

+ sensor: 输入, AI/BI, 传感器
+ cmd: 输出, AO/BO, 执行器, 指令
+ sp: 设定点, 内部控制变量, 时间表

所有的点必须使用 siteRef 标签和站点相关联，使用 equipRef 标签与特定的设备相关联。 如果某一点没有物理设备关系，则使用虚拟设备实体对逻辑分组进行建模。

按照惯例，使用多个标签对一个点的作用进行建模：

By convention multiple tags are used to model the role of a point:

+ where: discharge, return, exhaust, outside
+ what: air, water, steam
+ measurement: temp, humidity, flow, pressure

以下是AHU排气温度输入点的示例：
```
id: @whitehouse.ahu3.dat
dis: "White House AHU-3 DischargeAirTemp"
point
siteRef: @whitehouse
equipRef: @whitehouse.ahu3
discharge
air
temp
sensor
kind: "Number"
unit: "°F"
```

### 3.5.1 点的分类
点分为Bool，Number或Str三种类型，使用 kind 标签进行区分：

+ Bool: 以true或false来模拟数字量的点。Bool点也可以为文本定义一个枚举标签，用于true或false的状态
+ Number: 模拟诸如温度或压力等模拟量的点。这些点也应该包含 unit 用以指示该点的测量单位。
+ Str: 用“关闭，慢速，快速”等模式对枚举点进行建模。枚举点也应该定义一个 enum 标签。

### 3.5.2 Point Min/Max
The following tags may be used to define a minimum and/or maximum for the point:

+ minVal: minimum point value
+ maxVal: maximum point value

When these tags are applied to a sensor point, they model the range of values the sensor can read and report. Values outside of these range might indicate a fault condition in the sensor.

When these tags are applied to a cmd or sp, they model the range of valid user inputs when commanding the point.

### 3.5.3 Point Cur
The term cur indicates synchronization of a point's current real-time value. By real-time we typically mean freshness within the order of of a few seconds. If a point supports a current or live real-time value then it should be tagged with cur tag.

The following tags are used to model the current value and status:

+ curVal: current value of the point as Number, Bool, or Str
+ curStatus: ok, down, fault, disabled, or unknown
+ curErr: error message if curStatus indicated error

### 3.5.4 Point Write
Writable points are points which model an output or setpoint and may be commanded. Writable points are modeled on the BACnet 16-level priority array with a relinquish default which effectively acts as level 17. Writable points which may be commanded by the pointWrite operation should be tagged with the writable tag.

The following levels have special behavior:

+ **Level 1:** highest priority reserved for emergency overrides
+ **Level 8:** manual override with ability to set timer to expire back to auto
+ **Default:** implicitly acts as level 17 for relinquish default

The priority array provides for contention resolution when many different control applications may be vying for control of a given point. Low level applications like scheduling typically control levels 14, 15, or 16. Then users can override at level 8. But a higher levels like 2 to 7 can be used to trump a user override (for example a demand response energy routine that requires higher priority).

The actual value to write is resolved by starting at level 1 and working down to relinquish default to find the first non-null value. It is possible for all levels to be null, in which case the overall write output is null (which in turn may be auto/null to another system). Anytime a null value is written to a priority level, we say that level has been set to auto or released (this allows the next highest level to take command of the output).

The following tags are used to model the writable state of a point:

+ writeVal: this is the current "winning" value of the priority array, or if this tag is missing then the winning value is null
+ writeLevel: number from 1 to 17 indicate the winning priority array level
+ writeStatus: status of the server's ability to write the last value to the output device: ok, disabled, down, fault.
+ writeErr: indicates the error message if writeStatus is error condition

### 3.5.5 Point His
If a point is historized this means that we have a time-series sampling of the point's value over a time range. Historized points are sometimes called logged or trended points. Historized points should be tagged with the his tag.

Historized points can have their time-series data read/write over HTTP via the hisRead and hisWrite operations.

If a point implements the his tag, then it should also implement these tags:

+ tz: all historized points must define this tag with their timezone name (must match the point's site timezone)
+ hisInterpolate: optionally defined to indicate whether the point is logged by interval of change-of-value
+ hisTotalized: optionally defined to indicate a point is collected an ongoing accumulated value

The current status of historization is modeled with:

+ hisStatus: ok, down, fault, disabled, pending, syncing, unknown
+ hisErr: error message if hisStatus indicated error

## 3.6 Weather
Building operations and energy usage are heavily influenced by weather conditions. This makes modeling of weather data a critical feature of Project Haystack. Because weather stations and measurements are often shared across multiple buildings, weather is not modeled as part of a site. Rather the weather tag models a separate top-level entity which represents a weather station or logical grouping of weather observations.

All weather entities should define a tz tag. Optionally they can also define geolocation tags such as geoCountry, geoCity, and geoCoord.

### 3.6.1 Weather Points
Weather data follows the same conventions as points, but to indicate that they associated with a weather entity, and not a site entity, we use the special tag weatherPoint to indicate a weather related point.

The following weather points are defined by the standard library:

+ weatherCond: enumeration of conditions (clear, cloudy, raining)
+ air temp: dry bulb temperature in °C or °F
+ wetBulb temp: web bulb temperature in °C or °F
+ apparent temp: preceived "feels like" temperature in °C or °F
+ dew temp: temperature in °C or °F below which water condenses
+ humidity: percent relative humidity
+ barometric pressure: atmospheric pressure in millibar or inHg
+ sunrise: historized trend of sunrise/sunsets as true/false transitions
+ precipitation: amount of water fall in mm or inches
+ cloudage: percentage of sky obscurred by clouds
+ solar irradiance: amount of solar energy in W/m²
+ wind direction: measured in degrees
+ wind speed: flow velocity measured in km/h or mph
+ visibility: distance measured in km or miles

Weather points are associated with their weather entity using the weatherRef tag.

### 3.6.2 Weather Example
Here is an example of a weather station and its associated points:
```
id: @weather.washington
dis: "Weather in Washington, DC"
weather
geoCoord: C(38.895, -77.036)

id: @weather.washington.temp
dis: "Weather in Washing, DC - Temp"
weatherRef: @weather.washington
weatherPoint
point
temp
sensor
kind: "Number"
unit: "°F"

id: @weather.washington.humidity
dis: "Weather in Washing, DC - Humidity"
weatherRef: @weather.washington
weatherPoint
point
humidity
sensor
kind: "Number"
unit: "%RH"
```

### 3.6.3 Weather vs Outside Tags
We often model both local weather sensors and data from an official weather station. Local sensors are typically used for HVAC control sequences. But we might use official weather data for checking local sensor calibration or baseline energy normalization. In Haystack, weather station data is annotated with weatherPoint and site-local sensors with outside:

+ weatherPoint temp versus outside temp
+ weatherPoint humidity versus outside humidity


