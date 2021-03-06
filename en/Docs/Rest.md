# 12 Rest
## 12.1 Overview
The Haystack REST API defines a simple mechanism to exchange haystack tagged data over HTTP.

A haystack REST server implements a set of ops or operations. An operation is a URI that receives a request and returns a response. Standard operations are defined to query the database, setup subscriptions, or read/write history time-series data. Operations are pluggable so that vendors can enhance their REST interface with their own customized, value-added functionality.

Both requests and responses are modeled as grids. Grids are encoded using one the standard MIME types for grid serialization, or may be pluggable using HTTP content negotiation.

Technically the Haystack REST API isn't all that "RESTful" from a purist perspective - the ops design is more akin to a RPC model. But we use the term REST to distinguish the design from traditional WS-* web services that use XML, SOAP, etc (although this design could easily tunnel through that technology).

## 12.2 Authentication
Compliant HTTP implementations must implement the authentication protocol specified in the Auth chapter.

## 12.3 URI Namespace
A haystack server defines a HTTP URI as its base address. Operations are then mapped as path names under that address. Example:
```
http://server/haystack/           // base URI
http://server/haystack/{op}       // operation URI pattern
http://server/haystack/about      // about op
http://server/haystack/read       // read op
```

The base URI must be assumed to end with a trailing slash even if it is not always expressed that way.

## 12.4 Requests
A client makes a request to a server by sending a single grid and the server responses with a grid. Typically grids are encoded using Zinc, but the actual encoding used is pluggable using content negotiation.

### 12.41 GET Requests
Many operations require no grid argument or a grid with a single row. In this case, a request may be performed using a HTTP GET request. The request grid is encoded in the HTTP query string where each query parameter is mapped to tags in a single row. The tag value must be Zinc encoded, otherwise it is assumed to be of a Str type.

Example of request with empty grid:
```
// request URI
/haystack/about

// request grid in Zinc
ver:"3.0"
empty
```

Example of request with single Str tag:
```
// request URI
/haystack/read?filter=site

// request grid in Zinc
ver:"3.0"
filter
"site"
```

Example of request with multiple tags encoded as Zinc:
```
// request URI
/haystack/hisRead?id=@hisId&range=yesterday

// request grid in Zinc
ver:"3.0"
id,range
@hisId,"yesterday"
```

### 12.4.2 POST Requests
If the request grid is anything other than a single row of name/value pairs, then it must be be sent using HTTP POST. The client must encode the grid using a MIME type supported by server. The client can query the supported MIME types using the formats op. The following is an example of posting to the hisRead op using Zinc:
```
POST /haystack/hisRead HTTP/1.1
Content-Type: text/zinc; charset=utf-8
Content-Length: 39

ver:"3.0"
id,range
@outsideAirTemp,"yesterday"
```

## 12.5 Responses
If the request grid is successfully read by the server, then it processes the operation and returns the HTTP status code 200 and serializes the response result as a MIME encoded grid.

## 12.6 Error Handling
There are three type of errors which may occur when a client makes a request to a server:
+ Network I/O errors
+ HTTP errors
+ Request errors

Network errors occur when the client cannot make a successful TCP connection to the server. This might include invalid host name, port number, or network outage. Typically these sorts of errors are raised as I/O exceptions by the client's runtime.

HTTP errors occur when TCP connections can be successfully established, but the server cannot successfully handle the URI or request grid at the HTTP layer. The following HTTP status codes must be used in these cases:

+ 400: the client failed to specify a required header such "Content-Type"
+ 403: the client is not authorized to access the op
+ 404: the URI does not map to a valid operation URI
+ 406: the client "Accept" header requested an supported MIME type
+ 415: the client posted the request grid using an supported MIME type
+ 501: the HTTP method is other than "GET" or "POST"

Request errors occur after the server has successfully read the request grid. If the server cannot fulfill the request for any reason, then it must return HTTP response code of 200 with a error grid as the body. HTTP error codes must only be used if the request grid itself cannot be read. Request errors include but are not limited to:

+ invalid request grid data columns
+ invalid request grid data types
+ unknown or invalid entity identifiers
+ inconsistent data types or data values
+ server internal errors/exceptions

## 12.7 Error Grid
If a operation failed after a request grid is successfully read by a server, then the server returns an error grid. An error grid is indicated by the presence of the err marker tag in the grid metadata. All error grids must also include a dis tag in the grid metadata with a human readable descripton of the problem. If the server runtime supports stack traces, this should be reported as a multi-line string via the errTrace tag in the grid metadata.

Example of an error grid encoded as Zinc:
```
ver:"3.0" err dis:"Cannot resolve id: badId" errTrace:"UnknownRecErr: badId\n  ...."
empty
```

Clients must always check for the present of the err grid marker tag to determine if the reponse is an error or a valid result.

## 12.8 Content Negotiation
The default for all REST operations is to return the result as a grid (or grids) with the MIME type "text/plain" formatted using Zinc. You can request to receive the results in alternate formats by specifying the "Accept" header in your HTTP request.

The following "Accept" header MIME types are standardized:

+ Zinc: text/plain, text/zinc, */*, or unspecified
+ Csv: text/csv
+ Json: application/json

If you specify an "Accept" header for an unsupported MIME type, then the 406 Unacceptable error code is returned.

Example for reading all site entities as CSV:
```
GET /haystack/read?filter=site
Content-Type: text/plain; charset=utf-8
Accept: text/csv

Response:

HTTP/1.1 200 OK
Content-Type: text/csv; charset=utf-8

dis,area,geoAddr
Site A,2000ft²,"1000 Main St,Richmond,VA"
Site B,3000ft²,"2000 Cary St,Richmond,VA"
```

All "text/" media type must be be encoded using UTF-8.

## 12.9 Watches
Haystack adopts the watch design from oBIX to handle real-time subscriptions. Watches are a stateful polling mechanism designed to work efficiently over HTTP.

The life cycle of a watch is as follows

1. The client creates a new watch using the watchSub operation
2. The client polls for changed entities using the watchPoll operation
3. The client explicitly closes the watch using watchUnsub or the server may automatically close the watch if the client fails to poll after a lease period

During the lifetime of a watch, the client may optionally:

1. add new entities to the watch using watchSub
2. remove entities from the watch using watchUnsub
3. perform a poll refresh to re-read all the watched entities using watchPoll