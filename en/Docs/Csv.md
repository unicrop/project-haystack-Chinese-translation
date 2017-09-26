# 11 Csv
## 11.1 Overview
CSV stands for comma separated values. It is a plain text format commonly used for serialization of tabular data. It is specified in [RFC 4180](). CSV provides a simple way to get tabular data into other applications such as Excel.

## 11.2 Grid Format
Haystack defines a standard mapping of grids into CSV which is used by the [REST API](). Due to its simplicity, CSV does not provide full fidelity with the Haystack grid model -- metadata and type information is discarded.

The Grid to CSV mapping is as follows:

+ First row is the column display names (programmatic names and meta is discarded)
+ Subsequent rows map to rows from the Grid
+ Marker cells are encoded as the Unicode checkmark code point U+2713
+ Null cells are encoded as the empty string
+ Ref cells are encoded as "@id dis", for example "@3ef7 Site-1"
+ Bools are encoded as "true" or "false"
+ Strs and Uris are encoded using their unescaped value
+ Everything else is encoded using its [Zinc]() encoding

Example:
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