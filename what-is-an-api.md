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