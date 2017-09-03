# 12 Rest
## 12.1 概述
Haystack REST API定义了一种通过HTTP协议交换Haystack标签数据的简单机制。

Haystack REST 服务实现了一系列 ops 或 operations 操作。其中一种操作是接收请求并返回响应数据的URI。这些标准的操作可以用来查询数据库，设置订阅或读写基于时间序列的历史数据。这些操作是可插拔的，以便软件提供方可以通过自己定制的增值功能增强其REST接口。

请求和响应都被建模为Grid（二维表）。Grid使用一种用于网格序列化的标准MIME类型进行编码，或者使用可插拔的HTTP内容协商。

从技术上讲，Haystack REST API并不是纯粹意义上的“RESTful”，其 ops 设计更像一种RPC模型。但是我们使用术语REST将本设计与使用XML，SOAP等传统WS-* Web服务进行区分（尽管这种设计也可以轻松地通过这些技术实现）。


## 12.2 认证
兼容的HTTP必须实现在本文档中认证章节指定的认证协议。

## 12.3 URI命名空间
Haystack服务定义了一个基础HTTP URI地址，并将所有操作映射为该地址下的路径名称。例如：
```
http://server/haystack/           // 基础 URI
http://server/haystack/{op}       // operation URI 模式
http://server/haystack/about      // about op
http://server/haystack/read       // read op
```

基础URI地址必须以斜杠符号“/”结尾，即使通常并不以这种方式来表示。

## 12.4 请求
客户端通过一个Grid数据向服务器发出请求，服务器返回一个Grid数据。 通常Grid使用Zinc编码，但使用的实际编码可以通过内容协商进行插入。

### 12.41 GET 请求
许多操作不需要Grid参数或者单行数据的Grid。 在这种情况下，可以使用HTTP GET方法来执行请求。请求的Grid数据被编码在HTTP查询字符串中，其中每个查询参数与单行Grid标签形成映射关系。标签值必须是Zinc编码的，否则会被当做是Str类型。

使用空grid请求示例：
```
// 请求URI
/haystack/about

// 使用Zinc编码的grid请求
ver:"3.0"
empty
```

单个Str标签的请求示例：
```
// request URI
/haystack/read?filter=site

// 使用Zinc编码的grid请求：
ver:"3.0"
filter
"site"
```

使用多个标签编码为Zinc的请求示例：
```
// 请求 URI
/haystack/hisRead?id=@hisId&range=yesterday

// 编码为Zinc的gird请求
ver:"3.0"
id,range
@hisId,"yesterday"
```

### 12.4.2 POST 请求
如果请求Gird不是单行键值对，就必须使用HTTP POST发送请求。客户端必须使用服务器支持的MIME类型对Grid进行编码。客户端可以使用格式化的op查询被支持的MIME类型。以下是使用Zinc通过POST请求hisRead op的示例：
```
POST /haystack/hisRead HTTP/1.1
Content-Type: text/zinc; charset=utf-8
Content-Length: 39

ver:"3.0"
id,range
@outsideAirTemp,"yesterday"
```

## 12.5 响应
如果请求Gird数据由服务器成功读取，服务器将处理该操作并返回HTTP状态代码200，并把响应结果序列化为MIME编码的Grid。

## 12.6 错误处理
当客户端向服务器发出请求时，可能会出现三种类型的错误：
+ 网络I/O错误
+ HTTP错误
+ 请求错误。

当客户端无法成功向服务器发起TCP连接时，会发生网络错误。可能的原因包括无效的主机名，端口号或网络中断。通常这些错误由客户机运行时的I/O异常引起。

HTTP错误发生在TCP连接可以成功建立，但服务器无法在HTTP层面成功处理URI或Gird数据请求时。在这些情况下必须使用以下HTTP状态代码：

+ 400: 客户端没有指定正确的HTTP头，如”Content-Type”
+ 403: 客户端没有权限访问该操作
+ 404: URI未映射到有效的操作地址
+ 406: 无法返回客户端”Accept”头所要求的MIME类型
+ 415: 请求的格式不受请求页面的支持（* 有疑问）
+ 501: 服务器不具备完成请求的功能，如Http请求不是GET或POST方法

请求错误发生在服务器成功读取请求Gird数据后。如果服务器由于任何原因无法满足请求，那么它必须返回200的HTTP响应代码，并以包含错误信息的Grid作为响应体。只有当请求Grid本身无法读取时，才能使用HTTP错误代码。 请求错误包括但不限于：

+ 无效的Grid数据列
+ 无效的Grid数据类型
+ 未知或无效的实体标识符
+ 不一致的数据类型或数据值
+ 服务器内部错误或异常
