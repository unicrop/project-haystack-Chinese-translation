# 13 Ops
## 13.1 Overview
This chapter defines the standardized operations of the Haystack REST API. Each operation is specifies the format and options of the request grid and response grid.

## 13.2 About
Haystack定义了供[REST API]()使用的网格到CSV的标准映射。由于它很简单，CSV不能为Haystack网格模型提供完整的保真度 -- 元数据和类型信息被丢弃。

The about op queries basic information about the server.

**Request:** empty grid

**Response:** single row grid with following columns:

+ haystackVersion: Str version of REST implementation, must be "3.0"
+ tz: Str of server's default [timezone]()
+ serverName: Str name of the server or project database
+ serverTime: current DateTime of server's clock
+ serverBootTime: DateTime when server was booted up
+ productName: Str name of the server software product
+ productUri: Uri of the product's web site
+ productVersion: Str version of the server software product
+ moduleName: module which implements Haystack server protocol if its a plug-in to the product
+ moduleVersion: Str version of moduleName

## 13.3 Ops
The ops op queries which operations are available on the server.

**Request:** empty grid

**Response:** grid where each row represents a single operation with the following columns:

+ name: Str name of the operation in the URI namespace
+ summary: Str short description of the operation

Example of response in Zinc:
```
ver:"3.0"
name,summary
"about","Summary information for server"
"ops","Operations supported by this server"
"formats","Grid data formats supported by this server"
"read","Read records by id or filter"
```

## 13.4 Formats
The formats op is used to query which MIME types are availble to read and write grids.

**Request:** empty grid

**Response:** grid where each row represents one supported MIME type with following columns:

+ mime: Str MIME type encoded as "mediaType/subType", these value must not include parameters. Any "text/" media type must be be encoded using UTF-8
+ receive: Marker tag if server can read this format in requests (client can POST this format)
+ send: Marker tag is server can write this format in responses (client can request response in this format)

Example of response in Zinc:
```
ver:"3.0"
mime,receive,send
"text/csv",,M
"text/plain",M,M
"text/zinc",M,M
```

## 13.5 Read
The read op is used to read a set of entity records either by their unique identifier or using a filter.

**Request (by filter):** a grid with a single row and following columns:

+ filter: required Str encoding of [filter]()
+ limit: optional Number which specifies maximum number of entities to return in response

**Request (by id):** a grid of one or more rows and one column:

+ id: a Ref identifier

**Response:** grid with a row for each entity read. If a filter read and no matches were found this will be an empty grid with no rows. If a read by id, then each row corresponds to the request grid and its respective row ordering. If an id from the request was not found, the response includes a row of all null cells.

Example of filter read request:
```
ver:"3.0"
filter,limit
"point and siteRef==@siteA",1000
```

Example of read by id with three identifiers:
```
ver:"3.0"
id
@vav101.zoneTemp
@vav102.zoneTemp
@vav103.zoneTemp
```

Example of a read response where an id is not found:
```
ver:"3.0"
id,dis,curVal
@vav101.zoneTemp, "VAV-101 ZoneTemp",74.2°F
N,N,N
@vav103.zoneTemp, "VAV-103 ZoneTemp",73.8°F
```

Note: a read operation on points returns the last known values for [curVal]() and [curStatus](). It does not force cur value refresh from downstream data sources. For those cases you must use the [watchSub]() operation.


## 13.6 Nav
The nav op is used navigate a project for learning and discovery. This operation allows servers to expose the database in a human-friendly tree (or graph) that can be explored.

**Request:** a grid with a single row and a navId column. If the grid is empty or navId is null, then the request is for the navigation root.

**Response:** a grid of navigation children for the navId specified by the request. There is always a navId column which indicates the opaque identifier used to navigate to the next level of that row. If the navId of a row is null, then the row is a leaf item with no children.

Navigation rows don't necessarily always correspond to records in the database. However, if the navigation row has an id column then it is safe to assume the row maps to a record in the database. Clients should treat the navId as an opaque identifier.


## 13.7 WatchSub
The watchSub operation is used to create new [watches]() or add entities to an existing watch.

If the entities subscribed are themselves proxies for external data sources, then this operation should perform a downstream data refresh. For example if a Haystack gateway is used to proxy BACnet points, then a watch subscription to the Haystack points might initiate a poll or COV subscription to the downstream BACnet points. It is an implementation detail whether the data refresh occurs synchronously or asynchronously. Clients should expect that the latest data might not be available until a subsequent watchPoll operation.

**Request:** a row for each entity to subscribe to with the an id column and Ref values. In addition the following grid metadata is specified:

+ watchDis: Str debug string required when creating a new watch
+ watchId: Str watch identifier which is required to add entities to existing watch. If omitted, the server must open a new watch.
+ lease: optional Number with duration unit for desired lease period (server is free it ignore)

**Response:** rows correspond to the current entity state of the requested identifiers using same rules as [read op](): each response row corresponds to the request grid and its respective row ordering. If an id from the request was not found, the response includes a row of all null cells. Grid metadata is:

+ watchId: required Str identifier of the watch
+ lease: required Number with duration unit for server assigned lease period

If the reponse is an [error grid](), then the client must assume the watch is no longer valid, and open a new watch.

It is possible that clients may use an id for the subscription which is not the server's canonical id (for example if multiple aliases might be used to reference an entity). The canonical id is the one returned by the server in the watchSub response. Servers must use this same id during watchPoll operations. Clients must not assume that the id used by the watchSub request is the same id used by the watchSub response and watchPoll responses; however, the order of rows in watchSub request/response is guaranteed to allow clients to perform a mapping.


## 13.8 WatchUnsub
The watchUnsub operation is used to close a [watch]() entirely or remove entities from a watch.

**Request:** a row with the id column and Ref values for each entity to unsubscribe (if the watch is not be closed). Grid metadata:
```
watchId: Str watch identifier
close: Marker tag to close the entire watch The request also .
```

**Response:** empty grid

If the reponse is an [error grid](), then the client must assume the watch is no longer valid, and open a new watch.


## 13.9 WatchPoll
The watchPoll operation is used to poll a [watch]() for changes to the subscribed entity records.

**Request:** grid metadata:

+ watchId: required Str identifier of the watch
+ refresh: Marker tag to request full refresh

**Reponse:** grid where each row correspondes to a watched entity. The id tag of each row identifies the changed entity and correlates to the id returned by watchSub response. Clients must assume no explicit ordering of the rows. If the poll was for changes only, only the changed entities since last poll are returned. If no changes have occurred, then an empty grid is returned. If the poll is a full refresh, then a row is returned for each entity in the watch (invalid identifiers are not be included).

If the reponse is an [error grid](), then the client must assume the watch is no longer valid, and open a new watch.


## 13.10 PointWrite


