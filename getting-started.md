# 入门指南

现在，你已经安装了Apigility了，是时候来参观一下它。

## 假设

本章节假设你按照[安装向导](http://vergil.cn/archives/3/)运行，可在URL `http://localhost:8888/` 访问到应用的根目录。

> ### 注意：文件系统权限
> Apigility Admin UI将会写入到应用的文件系统，特别是在module/和config/目录。因此，运行Web服务器的用户必须有权限写入到这些目录和这些目录的文件中。Apigility包含一些逻辑来检测权限设置是否正确，如果它检测到权限不正确会抛出一个警告对话框。如果你看到这样一个对话框，使用操作系统工具来设置相应的文件系统权限。
当部署一个基于Apigility应用时，你不需要这些目录的写入权限。你只需要在开发过程中的写入权限。

### 第一步
访问`http://locahost:8888/`，你将被重定向到`http://localhost:8888/apigility/welcome`，它看起来像这样：
![](https://apigility.org/apigility-documentation/img/intro-getting-started-welcome.png)

单击导航栏的“Admin”或“Get Started”按钮会跳转到Apigility管理界面：
![](https://apigility.org/apigility-documentation/img/intro-getting-started-settings.png)

## 创建一个API
现在是时候来创建你的第一个API。点击导航栏上的“APIs”进入“APIs”页面：
![](https://apigility.org/apigility-documentation/img/intro-getting-started-apis.png)

然后点击“+ Create New API”按钮，弹出“Create New API”对话框：
![intro-getting-started-new-api-modal](https://apigility.org/apigility-documentation/img/intro-getting-started-new-api-modal.png)

在这个练习中，我们将创建一个名为“Status”的API;输入的“API Name”，然后按“Create API”按钮。当它完成后，你会被带到一个API概览屏幕：
![](https://apigility.org/apigility-documentation/img/intro-getting-started-status-api-v1.png)

## 创建一个RPC服务
现在我们将创建我们的第一个服务。单击菜单项“RPC Services”。
![intro-getting-started-rpc-services](https://apigility.org/apigility-documentation/img/intro-getting-started-rpc-services.png)

现在点击“Create New RPC Service”按钮：
![](https://apigility.org/apigility-documentation/img/intro-getting-started-new-rpc-service.png)

在“RPC Service name”填写“Ping”，在“Route to match”填写“/ping”。然后点击“Create RPC Service”按钮去创建服务。
创建后，单击该栏目的服务名称，将其展开。
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-view.png)

单击“Fields”选项卡。

![intro-getting-started-ping-service-fields-view](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-fields-view.png)

如果你在该服务的彩色条盘旋，你会看到两个按钮出现，一个绿色的“edit”按钮，一个红色的“delete”按钮。点击“edit”按钮，这样我们就可以编辑的字段。
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-fields-edit.png)

在“Field name”输入值“ack”，然后选择“Create New Field”按钮。展开的新面板标有“ack”。
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-fields-ack.png)

在“Description”项输入“Acknowledge the request with a timestamp”，然后点击“Save Changes”。

现在选择“Documentation”选项卡。
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-documentation.png)
正如我们的“Fields”一样，将鼠标悬停在该服务的颜色栏，并选择绿色的“edit”按钮，这样我们就可以编辑该文档。

![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-documentation-edit.png)
添加服务描述和GET方法描述。完成后，点击“generate from configuration”按钮将会自动填写“Response Body”项。然后点击“Save”按钮保存。
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-documentation-verify.png)

目前，我们的服务不执行任何操作。让我们来改变一下。
打开文件`module/Status/src/Status/V1/Rpc/Ping/PingController.php`。并按照像以下截图那样来编辑它:
![](https://apigility.org/apigility-documentation/img/intro-getting-started-ping-service-controller.png)

重要的部分是在第`05`行，这会导入`ZF\ContentNegotiation\ViewModel`。和第`11`-`13`行，从控制器返回了一个视图模型。

> ### 控制器和视图模型
一个控制器代码接收传入的请求，并确定哪些代码来执行操作，然后传递给视图的信息是什么渲染。
>
视图模型是包含我们要在响应中提供的代表性返回数据的值对象。

在这段代码，我们要说的是，响应应包含当前时间戳的`ack`属性。

## 测试我们的RPC服务
现在是时候来测试RPC服务。你可以用能够通过HTTP通信的任何工具做到这一点。我们收藏的一些包括：

* [Postman](http://www.getpostman.com/) 可用于Chrome浏览器
* [RESTClient](http://restclient.net/) 可用于Firefox，Chrome和Safari。
* [cURL](http://curl.haxx.se/) 古老而普及的命令行工具
* [HTTPie](http://httpie.org/) 一款“cURL类似的工具”;把它看作成cURL带有内置语法高亮和简化的请求语法。

在文档中，我们将展示实际使用的HTTP请求和响应的请求。
我们执行一个`GET`请求服务，并指定我们想要的HTML：

    GET /ping HTTP/1.1
	Accept: text/html
    
这将返回一个错误：

	HTTP/1.1 406 Not Acceptable
	Content-Type: application/problem+json
	{
		"detail": "Cannot honor Accept type specified",
		"status": 406,
		"title": "Not Acceptable",
		"type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html"
	}

Apigility默认为[JSON](http://www.json.org/)。如果指定了`Accept`头不同的媒体类型，它会报告说，它不能处理它。 （不过，你可以稍后配置服务，以处理其他媒体类型。）

现在，让我们试着请求JSON：

	GET /ping HTTP/1.1
	Accept: application/json

这将会正常工作！

	HTTP/1.1 200 OK
	Content-Type: application/json
	{
		"ack": 1396560875
	}

现在让我们做一个`POST`请求：

	POST /ping HTTP/1.1
	Accept: application/json
	Content-Type: application/json
	{ "timestamp": 1396560875 }

Apigility报错：

	HTTP/1.1 405 Method Not Allowed
	Allow: GET

Apigility负责HTTP方法协商的为你服务。这意味着如果一个请求是通过一个你不允许的方法，它会报告给用户一个`405`状态码，同时报告允许通过`Allow`的方法的响应头。

你也可以问apigility方法允许通过`OPTIONS`要求：

	OPTIONS /ping HTTP/1.1
    
将响应：

	HTTP/1.1 200 OK
	Allow: GET

恭喜！你已经创建了第一个API和第一个服务！