REST服务教程
===========

创建一个REST服务
在这一章中，我们将创建一个简单的REST服务。

假设
----
本章假定你已经阅读并遵循[安装指南](http://vergil.cn/archives/3/)和[入门篇](http://vergil.cn/archives/13/)。如果你还没有，请继续之前做的。

你需要安装和配置`zfcampus/statuslib-example`模块来完成本教程。遵循这些步骤：

步骤1
-----

在你的应用程序的根目录，执行如下：

	$ php composer.phar require "zfcampus/statuslib-example:~1.0-dev"

步骤2
-----

编辑文件`config/application.config.php`并且添加`StatusLib`模块：

	array(
		'modules' => array(
		/* ... */
		‘StatusLib',
		),
		/* ... */
	)

步骤3
-----

创建一个php文件`data/statuslib.php`，返回一个数组：

	<?php
	return array();
确保该文件的web server用户可写。

步骤4
-----

编辑文件`config/autoload/local.php`，添加以下配置：

	return array(
		/* ... */
		'statuslib' => array(
		'array_mapper_path' => 'data/statuslib.php',
		),
	);

步骤5
-----

最后，你将需要一个有效的HTTP基本认证文件，通常为`htpasswd`。你可以生成一个使用[Apache提供的标准htpasswd工具](http://httpd.apache.org/docs/2.2/programs/htpasswd.html)，或者使用一个[在线htpasswd生成器](http://www.htaccesstools.com/htpasswd-generator/)。存储`htpasswd`文件在您的应用程序`data/htpasswd`。请注意你使用的凭据以便以后可以使用它们。

一旦这些步骤完成，继续教程。

术语
----

在Apigility的文档中，特别是在这一章中，使用以下术语：

* 实体（Entity）：
返回一个可寻址项。实体是一个URI的唯一标识符。

* 集合（Collection）：
一组可寻址的实体。通常情况下，集合中包含的所有实体都是相同类型的，并且共享相同的基本URI作为集合。

* 资源（Resource）：
接收传入的请求数据，一个对象确定一个集合或实体是否在URI被确定，并且确定用什么动作来执行。

* 相关链接（Relational Links）：
一个URI到具有所描述的关系资源相关的链接。允许你来描述不同的实体和集合之间的关系。以及直接链接到它们，使web客户端可以对这些关系进行操作。这些有时也超媒体的链接。

REST服务返回实体和集合，并提供相关的实体和集合之间的超媒体链接。资源对象的协调行动，并返回实体和集合。

创建一个REST服务
---------------

在本章中，我们将构建一个REST服务示例。

导航到“APIs”页面，然后在我们前面的章节中创建的“Status”API。下一步，选择“REST Services”菜单项。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-rest-services.png)

点击“Create New REST Service”按钮
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-new-rest-service.png)

这个对话框有两个选项卡。一个是创建“Code-Connected（代码链接）”服务，另一个是创建“DB-Connected（数据库链接）”服务

> ### Code-Connected vs DB-Connected services
> 当你创建一个代码连接服务，Apigility创建定义了REST服务的所有可用的各种操作存根（原文：stub "Resource" class）“资源”类。这些操作返回“405 Method Not Allowed”的响应，直到你填写你自己的代码。“代码连接”方面意味着你将提供执行你的API的实际工作中的代码; Apigility提供用于暴露该代码作为一个API接线。
>
数据库连接服务允许你指定一个数据库适配器和一个数据表。然后Apigility创建一个“虚拟”资源，它代表委托操作到底层的`Zend\Db\TableGateway\TableGateway`实例。换句话说，它更快速应用程序开发（rapid application development，RAD）或原型工具。

在这个练习，我们将创建代码连接。在“REST Service Name”输入“Status”，然后按“Create Code-Connected REST Service”。一旦服务成功创建后，点击上面写着“Status”的颜色栏将其展开并可以查看服务。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-status-settings.png)

Apigility提供了许多合理的默认值：

* 集合只允许`GET`（获取列表）和`POST`（创建新实体）操作。
* 实体允许`GET`（获取实体），`PUT`（替换实体），`PATCH`（执行部分更新）和`DELETE`（删除实体）操作。
* 如果你的集合支持分页操作，Apigility将限制为每页25个资源。
* Apigility创建一个基于服务名称的路由URI（例如，`/status[/:status_id]`）

> ### URI路由
> Apigility运行在Zend Framework 2 MVC架构之上，并且因此使用其[路由引擎](http://framework.zend.com/manual/2.3/en/modules/zend.mvc.routing.html)。
>
> 通过Apigility生成的路由都是所谓的“Segment”路由。它允许你：
>
> * 指定URI的可选部分，使用`[]`语法。
> * 指定命名参数使用`:varName`或`:var_name`匹配。
> * 指定的字面匹配;没有一个命名参数，或花括号（`{}`）内被认为是文字。
>
> 对于REST服务，该URI生成都强制性匹配其本身的字面。当由单独指定，解析为一个集合。在上面的例子中，这路径是`/Status`。也有一个命名参数的可选部分，称为“实体标识符”：[/:status_id]。这将会匹配如`/status/foo`, `/status/2`, `/status/96fa5ac9-3ae2-45b2-84d5-c346936be292`这样的URI。
>
> 注意：集合URI和指定的实体URI时才能制定实体之间的/，你不能以斜线（/）请求集合。
> 为什么？因为REST的宗旨是一个URI代表一个资源。如果我们允许一个斜线（/），我们会被允许多个URI来解析到相同的资源。


展开“REST Parameters”部分，你会看到名为“Hydrator Service Name”的字段和`Zend\Stdlib\Hydrator\ArraySerializable`的值。我们要改变它与我们`StatusLib`示例库工作。

将鼠标悬浮在该服务的标题栏上出现绿色的“edit”按钮。点击它然后展开“REST Parameters”部分。在“Hydrator Service Name”选项，选择Zend\Stdlib\Hydrator\ObjectProperty。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-settings-edit.png)

> ### Hydrators
> Hydrators是对象，允许构造一个关联数组到一个特定的对象类型，反之亦然。每个hydrator使用了不同的策略，如何做到这一点。Apigility使用默认的hydrator类型是
`ArraySerializable`它期望目的实现两个方法：
> 
> * `getArrayCopy()` 用于获取一个数组的副本
> * `exchangeArray($array)` 用于构造一个数组对象
>
> （这些方法的使用和PHP的`ArrayObject`一样！）
>
> `ObjectProperty` hydrator创建一个数组的副本时将获取对象的所有公共属性，和构造对象时从数组填充对象的公告属性。

在我们的示例中，`StatusLib`库提供自己的实例和集合类。通过点击标题栏展开“Service Class Names”面板，并且编辑“Entity Class”字段读取`StatusLib\Entity`和“Collection Class”字段读取`StatusLib\Collection`。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-settings-classes.png)

> ### Service Classes
> 当你创建一个代码连接的服务，Apigility为你生成了4个PHP类文件：
>
> * 一个实体类。
> * 一个集合类继承自Zend\Paginator\Paginator。这将会允许你提供分页结果集。
> * 一个资源类用于执行操作。
> * 一个工厂类用于创建资源。
>
> 你的代码可能已经确定了实体类和集合类，所以你可以自由地忽略Apigility创建的存根（stub）类（指上述Apigility生成的类）。但是注意：如果你最终版本的API，你可能会发现有特定版本的实体和集合类可能是有用的。因为它们可以让你的模型在每个特定的版本暴露出只有你想暴露的属性。

现在，选择屏幕底部的绿色“Save”按钮。
下一步，我们定义一些字段和我们的API文档。

定义我们的服务的字段
------------------

我们将定义字段：

* `message` - 状态信息。它必须是非空的，并且不超过140个字符。
* `user` - 用户提供的状态信息。它必须是非空的，并且满足一个正则表达式。
* `timestamp` - 整型的时间戳。它不是必须提交的，但如果提交，只能是数字。它总是在响应中返回。

我们也会有一个`id`的字段，但这只是为了展示。

导航到“Fields”选项卡，然后编辑服务（绿色的“Edit”按钮，可以通过鼠标悬停在服务的标题栏找到）
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-fields-edit.png)

在名为“Field name”的文本输入框输入“message”，然后按`回车`来创建新的字段。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-field.png)

现在，用同样的方法创建“user”和“timestamp”字段
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-all-fields.png)

点击标有“message”的绿色栏中任意位置来展开它。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-edit.png)

在“Description（描述）”中输入“A status message of no more than 140 characters.（状态信息不能超过140个字符）”。在“ValidationValidate Failure Message（验证失败消息）”中输入“A status message must contain between 1 and 140 characters.（状态信息必须包含1至140个字符）”。

将鼠标悬停在“Filters（过滤器）”栏并且按“Add Filter”按钮展示“Add Filter”表单。在选择框选择`Zend\Filter\StringTrim`（提示：输入“trim”以缩小选择范围）后按“Add Filter”按钮。按“Cancel”按钮删除表单或旁边的“Add Filter”按钮完成。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-filter-trim.png)

现在，将鼠标悬停在“Validators（验证器）”标题栏，然后按“Add Validator”按钮来展示“Add Validator”表单。在选择框选择`Zend\Validator\StringLength`（提示：输入“string”以缩小选择范围）后按“Add Validator”按钮。按“Cancel”按钮删除表单或旁边的“Add Validator”按钮完成。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-validator-length.png)

将鼠标悬停在`Zend\Validator\StringLength`标题栏展示“Add Option”按钮。点击它。在出现的选择框中选择`max`（最大值）。选中后就会出现新的表单。输入值“140”，然后点击“Add Option”按钮。按“Cancel”按钮删除表单或旁边的“Add Option”按钮完成。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-validator-max.png)

这个时候，你应该在你的屏幕上看到如下：
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-message-complete.png)

点击“message”字段标题栏来折叠它。

在这个时候，你有一个练习：

* 更新“user”字段：
	* 添加描述：“The user submitting the status message.”
	* 添加验证失败信息：“"You must provide a valid user.”
	* 添加`Zend\Filter\StringTrim`过滤器
	* 添加`Zend\Validator\Regex`验证器，给它一个`pattern`选项和值`/^(mwop|andi|zeev)$/`（如果需要可以随意替换或添加其他名字或昵称）
* 更新“timestamp”字段：
	* 添加描述：“he timestamp when the status message was last modified.”
	* 添加验证失败信息：“You must provide a timestamp.”
	* 切换“Required”标记为“No”
	* 添加`Zend\Validator\Digits`验证器

完成后，在屏幕的右下角按绿色的“Save Changes”按钮。完成后的“user”和“timestamp”字段看起来会像以下截图：
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-user-complete.png)
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-timestamp-complete.png)

让我们转移到文档。

文档
----

REST服务让你的文档不仅允许通过HTTP方法，而是通过HTTP方法为每个集合和实体。

用于记录一个REST服务的过程就像是我们在[入门指南](http://vergil.cn/archives/13/)学会的，但有两个不同之处：

* 你将需要为**每个**集合和实体的HTTP方法创建文档。
* 有些方法还预期会请求数据，所以你需要为请求数据创建文档。

假设你已经记录了你的字段。“generate from configuration（生成配置）”按钮是生成当前请求和响应的配置文件。你会发现，相应的负载包括`_links`和`_embedded`成员：这是因为在Apigility REST服务默认使用[超媒体应用语言](http://stateless.co/hal_specification.html)（Hypermedia Application Language， HAL）格式，它提供了一种为你链接到其他暴露出的服务，以及将它们嵌入的资源。（有关HAL的更多信息，请阅读我们的[HAL入门](https://apigility.org/documentation/api-primer/halprimer)）

你现在是练习记录集合和实体的操作：

* 提供“创建、操作，和获取状态信息”服务的文档
* 提供"状态信息的操作列表"的集合文档
	* 对于`GET`方法，把它描述为“获取状态信息的分页列表”
	* 对于`POST`方法，把它描述为“创建一个新的状态信息”
* 提供“操作和获取单个信息”的实体文档
	* 对于`GET`方法，把它描述为“获取状态信息”
	* 对于`POST`方法，把它描述为“更新状态信息”
	* 对于`PUT`方法，把它描述为“替换状态信息”
	* 对于`DELETE`方法，把它描述为”删除状态信息“

除了`DELETE`，对于每个操作，使用”generate from configuration（生成配置）“按钮来生成请求和相应配置。如果你愿意，你可以编辑它们，但这样做的话这个教程就没有必要了。

稍后我们将检查文档。现在，让我们继续来认证和授权。

认证和授权
---------

点击”Authorization“菜单项，弹出授权页面。你会看到一个服务和HTTP的表格，以及一个警告，表示我们还没有设定认证。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-authorization.png)

我们现在先忽略这些警告。表格的标题标有`POST`，`PATCH`，`PUT`和`DELETE`,表示所有服务暴露出的这些操作都需要授权。然后点击绿色的“Save”按钮。
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-authorization-complete.png)

选中的方法和服务都需要授权，这意味着他们无法访问，除非用户能提供有效的凭据。如果你尝试执行我们这时标记的操作，你会发现你不能执行：你会得到一个`403 Forbidden`的响应。

所以，接下来是加入身份认证。该警告框提供一个链接到“authentication screen”。点击它
![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-authentication.png)

在这个例子中，我们将使用HTTP基本（HTTP Basic）身份认证。因此，点击“HTTP Basic”按钮。

我们使用相同的值作为占位符：填写“Authentication Realm”的值为`api`，“Location of htpasswd file”的值为`data/htpasswd`。完成后，点击蓝色的“Save”按钮。

我们已经完成描述我们的API...但是我们还没有和我们的代码联系在一起呢！是时候做这些了。


定义资源
--------

你可以回忆一下，之前Apigility为我们创建了4个类，分别是实体（entity）、集合（collection）、资源（resource）、工厂（factory），和一个工厂初始化资源。是时候放一些代码在我们的资源类了。

Apigility提供版本控制。代码的命名空间也是版本化的。这个功能允许你并行运行多个版本的API。

在`module/Status/src/Status/V1/Rest/Status/StatusResource.php`找到我们的资源类文件并在编辑器打开它。

我们做出的第一个改变是导入`StatusLib\MapperInterface`类。在上面的`use`段添加一行`use StatusLib\MapperInterface;`。

	use StatusLib\MapperInterface;
	use ZF\ApiProblem\ApiProblem;
	use ZF\Rest\AbstractResourceListener.php;

下一步，我们给类定义一个`protected`属性`$mapper`。

	class StatusResource extends AbstractResourceListener
	{
    	protected $mapper;

创建一个构造函数，接收一个`MapperInterface`参数，并将其分配给`$mapper`属性。

	protected $mapper;
    public function __construct(MapperInterface $mapper)
    {
        $this->mapper = $mapper;
    }

现在，我们有了映射器组成，让我们来写一些方法。

* 替换`create()`方法的方法体为`return $this->mapper->create($data);`。
* 替换`delete()`方法的方法体为`return $this->mapper->delete($id);`。
* 替换`fetch()`方法的方法体为`return $this->mapper->fetch($id);`。
* 替换`fetchAll()`方法的方法体为`return $this->mapper->fetchAll();`。
* 替换`patch()`方法的方法体为`return $this->mapper->update($id, $data);`。
* 替换`update()`方法的方法体为`return $this->mapper->update($id, $data);`。

你会注意到我们并没有更新类中的所有方法。有几种方法可用于对列表的操作，我们并不确定这些操作。

我们如何获取`$mapper`到资源？为此，我们将修改我们的工厂。在编辑器打开文件`module/Status/src/Status/V1/Rest/Status/StatusResourceFactory.php`并修改它，所以它的内容如下（你应该只在`__invoke()`方法里面修改`return`）：

	<?php
	namespace Status\V1\Rest\Status;
	class StatusResourceFactory
	{
    	public function __invoke($services)
    	{
        	return new StatusResource($services->get('StatusLib\Mapper'));
    	}
	}

上面是一个工厂，使用了[Zend Framework 2 Service Manager]（http://framework.zend.com/manual/2.3/en/modules/zend.service-manager.intro.html）。当你的服务被请求选定，这个工厂将创建一个运行的`StatusResource`实例。在该方法中，我们把`Statuslib`模块中已经定义的的另一个服务将其注入到我们的`StatusResource`。

在这点上，我们终于有一个可工作的REST服务！

让我们执行一些测试来看看它是如何工作的。


测试
----

第一个测试我们将`GET`请求到集合

	GET /status HTTP/1.1
	Accept: application/json

我们还没有添加任何状态消息，所以我们得到了一个空集合的响应。

	HTTP/1.1 200 OK
	Content-Type: application/hal+json
	{
    	"_embedded": {
        	"status": []
    	},
    	"_links": {
        	"self": {
            	"href": "http://localhost:8888/status"
        	}
    	},
    	"page_count": 0,
    	"page_size": 25,
    	"total_items": 0
	}

让我们尝试添加一些项：

	POST /status HTTP/1.1
	Accept: application/json
	Content-Type: application/json
	{
    	"message": "First post!",
    	"user": "mwop"
	}

还记得我们是如何配置认证和授权吗？好了，现在可以看到它的工作！

	HTTP/1.1 403 Forbidden
	Content-Type: application/problem+json
	{
    	"detail": "Forbidden",
    	"status": 403,
    	"title": "Forbidden",
    	"type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html"
	}

让我们提供凭证。如果你使用的HTTP客户端，如cURL或HTTPie，或REST客户端或Postman，这一般会允许你执行你的凭据，然后把它们变成一个`Authorization`头。你会看到下面是一个如下面的`Basic` token。

    POST /status HTTP/1.1
    Accept: application/json
    Authorization: Basic bXdvcDptd29w
    Content-Type: application/json
    {
        "message": "First post!",
        "user": "mwop"
    }

终于成功了！

    HTTP/1.1 201 Created
    Content-Type: application/hal+json
    {
        "_links": {
            "self": {
                "href": "http://localhost:8888/status/3c10c391-f56c-4d04-a889-bd1bd8f746f0"
            }
        },
        "id": "3c10c391-f56c-4d04-a889-bd1bd8f746f0",
        "message": "First post!",
        "timestamp": 1396709084,
        "user": "mwop"
    }

> 注意：每个实体的标识符是唯一的，当你创建一个饿新的状态信息将会有不同的标识符。

让我们找回status消息；我们可以使用URI在`self`关系链接里找到它：

	GET /status/3c10c391-f56c-4d04-a889-bd1bd8f746f0 HTTP/1.1
	Accept: application/json

    HTTP/1.1 200 OK
    Content-Type: application/hal+json
    {
        "_links": {
            "self": {
                "href": "http://localhost:8888/status/3c10c391-f56c-4d04-a889-bd1bd8f746f0"
            }
        },
        "id": "3c10c391-f56c-4d04-a889-bd1bd8f746f0",
        "message": "First post!",
        "timestamp": 1396709084,
        "user": "mwop"
    }

如果回到我们的集合URL，会有什么呢

	GET /status HTTP/1.1
	Accept: application/json

    HTTP/1.1 200 OK
    Content-Type: application/hal+json
    {
        "_embedded": {
            "status": [
                {
                    "_links": {
                        "self": {
                            "href": "http://localhost:8888/status/3c10c391-f56c-4d04-a889-bd1bd8f746f0"
                        }
                    },
                    "id": "3c10c391-f56c-4d04-a889-bd1bd8f746f0",
                    "message": "First post!",
                    "timestamp": 1396709084,
                    "user": "mwop"
                }
            ]
        },
        "_links": {
            "first": {
                "href": "http://localhost:8888/status"
            },
            "last": {
                "href": "http://localhost:8888/status?page=1"
            },
            "self": {
                "href": "http://localhost:8888/status?page=1"
            }
        },
        "page_count": 1,
        "page_size": 25,
        "total_items": 1
    }

让我们来更新status;发送PATCH请求更改消息：

	PATCH /status/3c10c391-f56c-4d04-a889-bd1bd8f746f0 HTTP/1.1
	Accept: application/json
	Content-Type: application/json
	{"message": "[Updated] First Post!"}

哎呀!这种方法需要认证!

    HTTP/1.1 403 Forbidden
    Content-Type: application/problem+json
    {
        "detail": "Forbidden",
        "status": 403,
        "title": "Forbidden",
        "type": "http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html"
    }

发送你的凭据：

    PATCH /status/3c10c391-f56c-4d04-a889-bd1bd8f746f0 HTTP/1.1
    Accept: application/json
    Authorization: Basic bXdvcDptd29w
    Content-Type: application/json
    {"message": "[Updated] First Post!"}

成功！


    HTTP/1.1 200 OK
    Content-Type: application/hal+json
    {
        "_links": {
            "self": {
                "href": "http://localhost:8888/status/3c10c391-f56c-4d04-a889-bd1bd8f746f0"
            }
        },
        "id": "3c10c391-f56c-4d04-a889-bd1bd8f746f0",
        "message": "[Updated] First post!",
        "timestamp": 1396709724,
        "user": "mwop"
    }

`PUT`操作类似于`PATCH`，通常它是用来提供一个完整的`替换`的实体。现在我们不会展示它。

让我们尝试一下`DELETE`请求。回想一下，它是需要授权的。所以我们第一时间提供凭据。

    DELETE /status/3c10c391-f56c-4d04-a889-bd1bd8f746f0 HTTP/1.1
    Accept: application/json
    Authorization: Basic bXdvcDptd29w

这将导致：

	HTTP/1.1 204 No Content

让我们看一下文档。


文档
----

在前一章和这里，我们创建了文档。如果你在Admin UI里看过，你可能会看到嵌入在每个服务的文档。然而，你还可以看文档本身。

在顶部导航是一项名为“API Docs”点击它。

![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-api-docs.png)

正如API的文档版本，并且可以为每个版本分别查看文档。点击“v1”链接。

![](https://apigility.org/apigility-documentation/img/intro-first-rest-service-api-docs-api.png)

你可以展开每个服务操作。展开一个操作显示您的请求和响应的细节操作，包括允许`Accept`和`Content-Type`请求头，预期`Content-Type`响应，预期响应状态码，等等。


概要
----

在这一章,我们讨论了:

* 创建一个REST服务。
* 给服务字段创建过滤器和验证器,其中包括为他们提供配置。
* 给你的服务写文档
* 为你的服务提供身份验证和授权。

Apigility是一个强大和灵活的工具，为你定义API，以及暴露它们，并提供一个工作流从创建到提供文档。你已经接触到表面，现在是时候去探索它可以为你怎么构建！