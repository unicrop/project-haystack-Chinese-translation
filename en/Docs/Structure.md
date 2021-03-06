# 3 Structure
## 3.1 Overview
The primary structure of the Haystack model is based on a hierarchy of three entities:

+ Site: single building with its own street address
+ Equip: physical or logical piece of equipment within a site
+ Point: sensor, actuator or setpoint value for an equip

This three level hierarchy defines the primary entities which are used in across projects. Other core entities include:

+ Weather: outside weather conditions

The following diagram illustrates this basic three level hierarchy and how they cross-reference each other:

![](siteEquipPoint.png)

## 3.2 Containment
Haystack is not based on a "tree structure" per se, however tree structures can be defined using reference tags. Since a given entity can have multiple reference tags, it easy to define multi-dimensional tree structures.

The core site/equip/point model is used as the primary tree structure and basic framework. However, alternate structures can be equally important for analytics:

+ Electrical Distribution: how are meters, submeters, and electrical loads related?
+ Air Distribution: how are AHUs, VAVs, and zones related?
+ Chilled Water/Steam Distribution: how are AHUs, central plants, boilers, and chillers related?

Due to the complexity of the domain, you should not assume that any one tree structure can be used to fully describe a building and its equipment. It is better to think of a Haystack project as a data graph where entities have multiple relationships defined using reference tags.

## 3.3 Site
A site entity models a single facility using the site tag. A good rule of thumb is to model any building with its own street address as its own site. For example a campus is better modeled with each building as a site, versus treating the entire campus as one site.

Core tags used with sites:

+ geoAddr: the geographic free-form address of the site (which might include other geolocation tags such as geoCity or geoCoord)
+ tz: the timezone where the site is located
+ area: square footage or square meters of the facility. This enables site normalization by area.
+ weatherRef: associate the site with a weather station to visualize weather conditions and perform weather based energy normalization
+ primaryFunction: enumerated string which describes the primary function of the building
+ yearBuilt: four digit year in which the building was constructed

Here is an example of a site entity fully tricked out with geolocation tags:
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

## 3.4 Equip
Equipment is modeled using the equip tag. Equipment is often a physical asset such as an AHU, boiler, or chiller. However, equip can also be used to model a logical grouping such as a chiller plant.

All equipment should be associated within a single site using the siteRef tag. In turn, equipment will often contain points which are are associated with the equipment via the equipRef tag.

Here is an example of a AHU equipment entity:
```
id: @whitehouse.ahu3
dis: "White House AHU-3"
equip
siteRef: @whitehouse
ahu
```
The equipRef tag can optionally be used on equip entities to model nested equipment and containment relationships.

## 3.5 Point
Points are typically a digital or analog sensor or actuator entity (sometimes called hard points). Points can also represent a configuration value such as a setpoint or schedule log (sometimes called soft points). Point entities are tagged with the point tag.

All points are further classified as sensors, commands, or setpoints using one of the following three tags:

+ sensor: input, AI/BI, sensor
+ cmd: output, AO/BO, actuator, command
+ sp: setpoint, internal control variable, schedule

All points must be associated with a site via the siteRef tag and a specific piece of equipment via the equipRef tag. If a point doesn't have physical equipment relationship, then use a virtual equip entity to model a logical grouping.

By convention multiple tags are used to model the role of a point:

+ where: discharge, return, exhaust, outside
+ what: air, water, steam
+ measurement: temp, humidity, flow, pressure

Here is an example of an AHU discharge air temperature input point:
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

### 3.5.1 Point Kinds
Points are classified as Bool, Number, or Str using the kind tag:

+ Bool: model digital points as true/false. Bool points may also define an enum tag for the text to use for the true/false states
+ Number: model analog ponts such as temperature or pressure. These points should also include the unit to indicate the point's unit of measurement.
+ Str: models an enumerated point with a mode such as "Off, Slow, Fast". Enumeraed points should also define an enum tag.

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


