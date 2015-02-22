About Authentication
==============

Authentication is the process by which *when* an identity is presented to the application, the
application can *validate the identity is in fact who they say they are*.  In terms of API's and
Apigility, identities are delivered to the application from the client through the use of the
`Authorization` request header.  This header, if present, is parsed and utilized in one of the
configured authentication schemes.  If no header is present, Apigility assigns a default identity
known as a "guest" identity.  The important thing to note here is that authentication is not
something that needs to be turned on because *it is always on*. It just needs to be configured to handle when
an identity is presented to Apigility.  If no authentication scheme is configured, and an identity
is presented in a way that Apigility cannot handle, or is not configured to handle, the "guest"
identity will be assigned.

Apigility delivers three methods to authenticate identities: HTTP Basic authentication, HTTP Digest
authentication, and OAuth2 (by way of Brent Shaffer's [PHP OAuth2
package](https://github.com/bshaffer/oauth2-server-php)).  HTTP Basic and HTTP Digest
authentication can be configured to be used with minimal tools.

Authentication is something that happens "pre-route", meaning it is something that is configured 
per-application instead of per-module/API that is configured.  So if you need per-API groups of 
users for your APIs, it might make sense to either break your APIs out into their own Apigility 
powered applications or use a more advanced code-driven extension to the authentication module.

To get started with any of the configurable authentication schemes, click "Settings", then 
"Authentication":

![Authentication settings](/asset/apigility-documentation/img/auth-authentication-settings.jpg)

Once here, you will be presented with the aforementioned authentication schemes to be configured.


关于授权
==============
*当*一个身份信息在应用中被提交，Authentication就能够处理它，这样可以 *确认当前访问者的真实身份。在API和Apigility的协议中，身份证明要存放在应用请求的头部。如果在请求头里提供了身份证明，将根据authentication的配置规则进行解释并使用。 否则Apigility将提供一个默认的身份证明，这称为”游客(guest)“。 需要注意的一点是authentication不需要手动判断在哪些情况下开启，因为它一直都处于 *开启* 状态。 你只需要配置好authentication的处理规则，如果没有的话，那么Apigility将不能验证请求的身份，转而将其标识为”游客“。

Apigility提供了3个方法来验证身份：HTTP Basic authentication, HTTP Digest
authentication, 和 OAuth2 (by way of Brent Shaffer's [PHP OAuth2
package](https://github.com/bshaffer/oauth2-server-php))(TODO::翻译并修正连接).  HTTP Basic 和 HTTP Digest
authentication can be configured to be used with minimal tools.