# 6 Grids

## 6.1 Overview

Grids are two-dimensional tabular representations of tagged entities. We use grids as the core data model to serialize haystack tagged data over HTTP using the [Rest]() API.

## 6.2 Structure
A grid is composed of:

+ Grid level metadata
+ One or more columns of a programmatic name and metadata
+ Zero or more Rows which are defined as scalar cells

Metadata is just a list of tags as name value pairs as specified by the [tag model](). Grid level metadata allows us to specify tags about the entire grid.

Columns and rows are used to model a list of Haystack tagged entities. Columns are computed by the union of all unique tag names in a list of entities. Each column is composed of a programmatic name which must be a valid [tag name](). Columns may also specify metadata using tags. The [dis]() column tag may be used to provide a human friendly display name for the column (since the programmatic name will likely be camel case).

Rows are used to model the list of tagged entities. Rows are composed of a cell for each column in the grid. Each cell must be a valid [tag kind]() scalar value or may be null/sparse if the row doesn't have a value for a given column.

## 6.3 Example
Consider three haystack entities which model sites:
```
id: @site-a
dis: "Site A"
site
area: 45000ft²

id: @site-b
dis: "Site B"
site

id: @site-c
dis: "Site C"
site
area: 62000ft²
phone: "(804) 555-1234"
```

Our three entities all have an id, dis, and site tags. In addition two have the area tag, and one has a phone tag. To combine these three entities into a grid we end up with five columns and three rows:
```
id       dis       site  area      phone
-------  -------   ----  --------  ------
@site-a  "Site A"  ✓     45000ft²
@site-b  "Site B"  ✓
@site-c  "Site C"  ✓     62000ft²  "(804) 555-1234"
```

Note the columns are union of all tags shared by the entities. Because not every entity shares the same columns, we have sparse or null cells. We could further add grid level or column level meta.