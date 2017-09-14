# 5 Units
## 5.1 Overview
All number tag values can be annotated with an optional unit. In addition, it is required to annotate each numeric point with the unit tag. In both cases, the unit must be an identifier defined by the standard unit database.

## 5.2 Unit System
As a general principle, all the data associated with a given site should exclusively use either the SI metric system or the US customary system. Mixing unit systems within one site will cause serious headaches.

## 5.3 Database
The unit database used by Project Haystack is managed by the Fantom open source community as part of the sys::Unit API. This database was originally based on the oBIX specification, but has since been expanded to allow multiple aliases to be used for each unit.

Each unit of measurement has a full name and zero or more symbols which are used as aliases for that unit. For example "square_meter" is the full name and the symbol alias is "m²". Some units might have multiple symbols, for example "hour" has the symbols "hr" and "h". Some units like "day" have no symbols.

All unit identifiers are limited to the following characters:

+ any Unicode char over 128
+ ASCII letters a - z and A - Z
+ underbar _
+ division sign /
+ percent sign %
+ dollar sign $

By convention the symbol is the preferred notation. If there are multiple symbols, then the last symbol defined by the database is the preferred one.

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

