关于授权
==============
*当*一个身份信息在应用中被提交，Authentication就能够处理它，这样可以确认当前访问者的真实身份。在API和Apigility的协议中，身份证明要存放在应用请求的头部。如果在请求头里提供了身份证明，将根据Authentication的配置规则进行解释并应用。 否则Apigility将注入一个已知的默认身份：”游客(guest)“。 需要注意的一点是Authentication不需要手动判断在哪些情况下开启，因为它一直都处于 *开启* 状态。 你只需要配置好authentication的处理规则，如果没有的话，那么Apigility将不能验证请求的身份，转而将其标识为”游客“。

Apigility提供了3个方法来验证身份：HTTP Basic, HTTP Digest 和 OAuth2 (可以参考Brent Shaffer的[PHP OAuth2
package](https://github.com/bshaffer/oauth2-server-php)).  HTTP Basic 和 HTTP Digest 能够在最精简的环境下使用.

Authentication is something that happens "pre-route", meaning it is something that is configured 
per-application instead of per-module/API that is configured.  So if you need per-API groups of 
users for your APIs, it might make sense to either break your APIs out into their own Apigility 
powered applications or use a more advanced code-driven extension to the authentication module.

需要配置Authentication, 可以点击 "Settings" - "Authentication":

![Authentication settings](/asset/apigility-documentation/img/auth-authentication-settings.jpg)

你可以按照上述方案进行设置