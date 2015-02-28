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
---------
含状态传输（英文：Representational State Transfer，简称REST）不是一个规范，而是围绕HTTP规范设计的架构。[维基百科](http://zh.wikipedia.org/wiki/REST)的文章提供了一个很好的REST的概念的概述，和丰富的资源。

REST利用HTTP的优势，建立在：
* 对资源URI作为唯一标识符。
* 丰富的设置HTTP动词对资源的操作。
* 为客户端指定表示它们可以呈现的格式的能力，并为服务端履行这些（或者如果它不能将表明）
* 资源来表示关系之间的联系。（如，超媒体链接。就像在HTML文档那样）

当谈论关于REST，[Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)经常被用来实现一个设计良好的REST API时，描述必要的关注。它包括4个层次，从0开始索引：

* **Level 0**: 远程过程调用使用HTTP作为传输机制。从本质上来讲，这正如前一节所述的RPC。HTTP被用来作为一个隧道机制，对所有服务通过一个单一入口点，并没有考虑HTTP的超媒体方面的优点：URI来表示独特的资源，HTTP动词，指定和返回多种媒体类型，或服务之间的链接能力。（注意：某些Level0服务将使用一个以上的HTTP动词，但通常在只是混乱使用`POST`（用于操作可能导致数据的更改）和`GET`（当仅读取数据）。）
* **Level 1**: 使用URI表示单个资源的服务。这区别于Level0是利用每个操作URI或“资源”（毕竟“URI”中的“R”代表“Rescource”！）。代替`/services` URI，你将会有每个服务之一：`/users`，`/contacts`，等等；此外，你可能会允许服务中的个别项通过唯一的URI：`/users/mwop`将获得“mwop”用户。然而，在Level1，你仍然没有良好的使用HTTP动词，不同的媒体类型，或者服务之间的链接。
* **Level 2**: 使用HTTP动词和头与资源的互动。在Level1的基础上，Level2服务开始全方位使用HTTP请求方法：`PATCH`，`PUT`，和`DELETE`被添加到兵工厂。`GET`用于不改变状态的安全操作，并且可以使用于使用任何次数得到相同的结果；也就是说，它使用了HTTP请求缓存。该服务返回相应的HTTP响应状态的错误，基于该错误类型；不再带有`200 OK`在错误请求的响应体。HTTP头可以用于不同的响应；例如，不同的表现形式，可以根据`Accept`头返回（并且使用`Accept`头请求替代表示,而不是文件扩展名，为了推动这一观点，相同的资源被操控的时候不管这些表现形式）。
* **Level 3**: 超媒体控制。大多数API设计者都可能达到Level2，并且感觉他们很好的完成他们的工作。然而，该API一个方面可以使它更加实用：链接资源。考虑一下：你进行请求到一个提供活动门票的API。在Level2，该响应可能是一个门票的列表；但是在Level3，更进一步了，它提供了链接到每张门票的资源，让你可以预订其中的任何一个。
链接的意义是告诉API的用户下一步可以做什么。一方面是它允许服务器更改资源URI，假设用户将遵循那些在服务端本身返回的链接。另一方面，链接帮助该API文档本身；用户可以遵循API中的链接了解如何使用各种相关资源，和他们能做什么。

实质上，一个好的REST API：
* 给服务和那些服务公开的项使用唯一的URIs。
* 使用全方位的HTTP动词，以对这些资源执行操作和允许不同的内容响应，启用HTTP级缓存，等等。
* 提供了资源的相关链接，告诉用户下一步可以做什么。

所有的这些理论有助于告诉我们REST服务应该如何行动，但很少告诉我们如何去实现它们。这是有点被设计；REST更像是一种架构考虑。然而，这意味着该API设计者现在必须作出大量的选择：

* 你将要暴露出什么样的表示格式？你将如何报告那些你不能完成对于给定的请求表示？REST没有规定任何具体的格式。
* 你将要如何报告错误？同样，REST没有规定任何具体的错误报告格式，超出这表明适当的HTTP响应代码应使用；然而，这些单独不提供足够的细节，以对客户有用的。
* 你将如何给资源宣传它可用的HTTP方法？如果客户使用不支持的请求方法，你会怎么做？
* 你将如何处理特性，比如身份验证？Web APIs通常是无状态的，并且也不应该依赖特性，如session cookie。用户将如何提供他们对每个请求的凭据？你将使用HTTP认证，或者OAuth2，说着创建API Token？
* 你将如何处理超媒体链接？一些格式，比如XML，基本上具有内置链接；其他的，比如JSON，没有原生格式，这意味着你需要选择你将如何提供链接。
* 你将如何记录哪些资源可用？不像RPC，REST服务没有“内置”的机制来描述；而超媒体的链接将协助，用户仍然需要知道各个入口点的API。

这些都不是琐碎的问题，并且在许多情况下，可以使一个选择影响你的另一个选择。

简单的说，大多数REST提供了另人难以置信的灵活性和力量，但需要你为客户提供坚实的，优质的体验做出很多选择。

Apigility的API类型
==================

RPC在Apigility
--------------

尽管在RPC部分列出的问题，Apigility提供RPC服务。但是，Apigility的RPC展示略有不同的特点：

* 一个单一的服务端点可以对应多个HTTP方法。请求配置以外的使用方法会导致`405 Method Not Allowed`状态；`OPTIONS`请求讲详述哪些请求方法是允许的。
* 一个单一的服务端点可以提供多种响应。默认情况下，我们返回JSON。
* 以一致的方式报告错误（特别的，[application/problem+json](https://github.com/VergilLai/ApigilityDoc-zh_cn/blob/master/error-reporting)）

我们把RPC看作是大量的一次性操作，或者有更多面向“动作”的操作（执行操作）而不是面向“资源”（操作“东西”）。

就像REST，如果需要，我们允许RPC服务提供多种响应表示，虽然我们默认为JSON（如果需要，你甚至可以执行超媒体链接）。


REST在Apigility
---------------

在Apigility，我们为REST服务进行以下选择:

* REST URIs提供“集合”和从这些集合单独可寻址的“实体”。每种类型可以指定允许HTTP请求方法；请求配置以外的使用方法会导致`405 Method Not Allowed`状态。`OPTIONS`请求讲详述哪些请求方法是允许的。
* 默认情况下，我们使用[超媒体应用程序语言（Hypermedia Application Language)](https://github.com/VergilLai/ApigilityDoczh_cn/blob/master/halprimer)，它不仅提供了关系的链接而且嵌入其他可寻址资源的能力。
* 以一致的方式报告错误（特别的，[application/problem+json](https://github.com/VergilLai/ApigilityDoc-zh_cn/blob/master/error-reporting)）

一个典型的REST URI看起来像`/status[/:status_id]`，请求到`/status`将返回一个集合。同时加入一个唯一值`status_id`,例如`/status/12345678`，将会返回一个status的实体。集合和实体都可以返回任意的关系链接到其他的资源。