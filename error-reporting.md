错误报告
========

HAL为资源带有关系链接定义一个通用的MediaType，的确是一个伟大的工作。然而，你如何报告错误？HAL在这个问题上保持沉默。

REST主张表明应该使用HTTP响应状态码，但没有做标准化格式的响应。

对于JSON的API，虽然，两种格式都开始实现大量采用：`application/vnd.error+json`和`application/problem+json`。Apigility为后者提供支持，[对于HTTP API的问题详细信息](http://tools.ietf.org/html/draft-nottingham-http-problem-06)。Apigility把它称之为`API Problem`并且通过[zf-api-problem ](https://github.com/zfcampus/zf-api-problem)模块提供支持。


API Problem
------------

API问题判断MediaType `application/problem+json`；有趣的是，还有一个XML变种，虽然此事Apigility不会对它提供支持。

一个API问题的载体具有以下结构：
* **type**: 一个URL到文档描述的错误条件（可选，“about:blank”假设没有提供，应解析为一个可读的文档；Apigility总是提供这个；）
* **title**: 一个简短的标题的错误条件（必需，对于每个问题，应该相同的**type**，Apigility总是提供这个）
* **status**: 当前请求的HTTP状态码（可选，Apigility总是提供这个）
* **detail**: 特定于该请求错误的细节（可选，Apigility）
* **instance**: URI识别这一问题的具体实例

一个载体的例子：

    HTTP/1.1 500 Internal Error
    Content-Type: application/problem+json
    {
        "type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html",
        "detail": "Status failed validation",
        "status": 500,
        "title": "Internal Server Error"
    }