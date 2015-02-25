什么是API？
==========

API的全称是“Application Programming Interface（应用程序编程接口）”，并作为一个术语，指定了软件如何交互。

一般来说，当我们今天提到API时更确切的说是**Web APIs**。那些通过超文本传输协议传送数据（HTTP）。对于这个特定的情况下，那么，一个API指定一个用户可以如何使用一个API公开的服务：什么URI可以使用，每个URI可以使用什么HTTP方法，它接受什么查询字符串参数，什么数据可以发送到请求主体，和用户可以得到什么预期的响应。

API的类型
---------

Web api可以分为两大类:

* **远程过程调用 (Remote Procedure Call, RPC)**
* **表述性状态传递 (REpresentational State Transfer, REST)**

RPC
---

RPC通常的特点是单个URI在其上的许多操作可以被调用,通常只通过`POST`。例如[XML-RPC](http://www.xmlrpc.com/) 和 [SOAP](http://www.w3.org/TR/soap/)。通常情况下，你会传递一个结构化的请求，包括要调用的操作名称和你想要传递给操作的参数；它将响应一个结构化的格式。

一个例子：

    POST /xml-rpc HTTP/1.1
    Content-Type: text/xml
    <?xml version="1.0" encoding="utf-8"?>
    <methodCall>
        <methodName>status.create</methodName>
        <params>
            <param>
                <value><string>First post!</string></value>
            </param>
            <param>
                <value><string>mwop</string></value>
            </param>
            <param>
                <value><dateTime.iso8601>20140328T15:22:21</dateTime.iso8601></value>
            </param>
        </params>
    </methodCall>

上面`POST`到一个已知的URI：`/xml-rpc`。请求载体包含要调用的操作：`status.create`，和参数传递给它。这些参数将会按照所定义的顺序被传递。在这种情况下，处理程序给定的操作在PHP可能像是这样：

    class Status
    {
        public function create($message, $user, $timestamp)
        {
            // do something...
        }
    }


RPC通常是非常程序性的;通常能很好地映射到函数定义的操作。

上面的可能响应如下：

    HTTP/1.1 200 OK
    Content-Type: text/xml
    <?xml version="1.0" encoding="utf-8"?>
    <methodResponse>
        <params>
            <param><value><struct>
                <member>
                    <name>status</name>
                    <value><string>First post!</string></value>
                </member>
                <member>
                    <name>user</name>
                    <value><string>mwop</string></value>
                </member>
                <member>
                    <name>timestamp></name>
                    <value><dateTime.iso8601>20140328T15:22:21</dateTime.iso8601></value>
                </member>
            </struct></value></param>
        </params>
    </methodResponse>


我们从响应中接收一个值。在这种特殊情况下，我们在返回值接收到结构，这大概相当于一个匿名对象或者关联数组。在PHP中，该值可能是这样的：

    array(
        'status' => 'First post!',
        'user' => 'mwop',
        'timestamp' => '20140328T15:22:21',
    )

当错误发生时,大多数RPC格式有一种标准的方式报告。在XML-RPC，这是一个“错误”的回应，而SOAP有一个SOAP错误。以XML-RPC作为例子，比方说，我们只传递了一个单一的值到服务上面，那么我们可能会得到如下所示的错误响应：

    HTTP/1.1 200 OK
    Content-Type: text/xml
    <?xml version="1.0" encoding="utf-8"?>
    <methodResponse>
        <fault><value><struct>
            <member>
                <name>faultCode</name>
                <value><int>422</int></value>
            </member>
            <member>
                <name>faultString</name>
                <value><string>
                    Too few parameters passed; must include message, user, and timestamp
                </string></value>
            </member>
            <member>
                <name>timestamp></name>
                <value><dateTime.iso8601>20140328T15:22:21</dateTime.iso8601></value>
            </member>
        </struct></value></fault>
    </methodResponse>

这里有一点要注意的是，RPC通常是在响应体里包含所有错误报告。HTTP状态码不会有所不同，这意味着你需要检查返回值，以确定是否发生了错误！

最后，很多RPC实现还通过协议本身提供文档给他们的最终用户。在SOAP，文档是[WSDL](http://www.w3.org/TR/wsdl)。在XML-RPC，文档是通过各种“[system.](http://tldp.org/HOWTO/XML-RPC-HOWTO/xmlrpc-howto-interfaces.html)”方法。这个自我说明功能（并不总是）可以在实现后提供给消费者如何与服务交互的宝贵信息。

要记住RPC的要点是：

* 一个服务端点，许多操作。
* 一个服务端点，一种HTTP方法（通常是`POST`）。
* 结构化的，可预测的请求格式。结构化的，可预测的响应格式。
* 结构化的，可预测的操作报告格式。
* 可用操作的结构化文档。

这一切说明，RPC往往是一个不太适合于web APIs：
* 你不能确定通过URI有多少资源是可用的。
* 缺乏HTTP缓存，无法使用原生HTTP动词于常用的操作；缺乏HTTP响应代码错误报告，需要对结果反思以确定是否发生了错误。
* “一刀切”的格式可以限制；客户消费替代序列化格式不能被使用，和消息格式通常对可以提交或返回的数据类型强加不必要的限制。

概括的说，大多数RPC变种通常在使用web APIs不使用HTTP充分发挥其功能。


REST
====
含状态传输（英文：Representational State Transfer，简称REST）不是一个规范，而是围绕HTTP规范设计的架构。[维基百科](http://zh.wikipedia.org/wiki/REST)的文章提供了一个很好的REST的概念的概述，和丰富的资源。

REST利用HTTP的优势，建立在：
* 对资源URI作为唯一标识符。
* 丰富的设置HTTP动词对资源的操作。
* 为客户端指定表示它们可以呈现的格式的能力，并为服务端履行这些（或者如果它不能将表明）
* 资源来表示关系之间的联系。（如，超媒体链接。就像在HTML文档那样）

当谈论关于REST，[Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)经常被用来实现一个设计良好的REST API时，描述必要的关注。它包括4个层次，从0开始索引：

* **Level 0**:
* **Level 1**:
* **Level 2**:
* **Level 3**: