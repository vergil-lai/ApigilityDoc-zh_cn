Authentication & Authorization
==============================

Apigility takes a lightweight, layered, and extensible approach to solving both problems of 
authentication and authorization.  In Apigility this infrastructure is already in place and ready to be 
configured to use, or for more advanced use cases, to be extended.  Many of these features can be 
explored through the Apigility user interface.

While much of the terminology might be similar, authentication and authorization **IS NOT** the same 
as the set of allowed HTTP methods.  These methods are labeled as _allowed_ in the sense that a particular 
REST or RPC service can respond to that method regarless of what authentication/authorization is 
configured, or which identity is present on any given request to that particular service; see [the section on HTTP negotiation](/api-primer/http-negotiation.md) for more information.


授权 & 认证
==============================
Apigility包含一组轻量级、分层、可扩展的方法来解决authentication(授权)和authorization(认证)问题。它们已经被配置好并随时可以使用，一些高级的功能可以使用扩展来实现。更多的特征可以参考Apigility用户接口。

虽然很多术语是相似的，但authentication/authorization跟HTTP方法(HTTP methods)**不同**。 后者被打上”允许“的标签，表示即使你在REST和RPC服务中指定了authentication/authorization参数（无论是什么配置），或者客户端在请求中提供了可能不合法的身份ID，这些HTTP方法都能够被正确的处理。要了解更多，请参考[the section on HTTP negotiation](/api-primer/http-negotiation.md)(TODO::翻译和修正连接)