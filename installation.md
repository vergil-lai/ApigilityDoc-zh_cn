安装
====

从终端安装：
-----------

安装`Apigility`最容易的方法是从你的终端，执行以下命令：

    curl -sS https://apigility.org/install | php
    
如果你没有安装cURL，你可以使用php本身：

	php -r "readfile('https://apigility.org/install');" | php

否则，你可以使用以下一个过程替代的`Apigility`安装。

通过发行版tar包：
----------------

从Apigility[下载](https://apigility.org/download)页面获取最新版本。
解压它：

	tar xzf zf-apigility-skeleton-1.0.0beta1.tgz

通过Composer (create-project)
-----------------------------

你可以从[Composer](http://getcomposer.org/)中使用`create-project`命令去创建项目：

	curl -s https://getcomposer.org/installer | php --
	php composer.phar create-project -sdev zfcampus/zf-apigility-skeleton path/to/install

通过Git(Clone)
--------------

首先，克隆代码库：

	git clone https://github.com/zfcampus/zf-apigility-skeleton.git # optionally, specify the directory in which to clone
	cd path/to/install

然后，你需要使用[Composer](http://getcomposer.org/)安装依赖关系：
假设你已安装了Composer：

	composer.phar install

所有的方法：
-----------

一旦你有了基本的安装，你需要把它放在开发模式：

	cd path/to/install
	php public/index.php development enable # put the skeleton in development mode

现在，启动它！请执行下列操作之一：

在你的web服务器创建虚拟主机，DocumentRoot指向项目的`public/`目录
在PHP内置web服务器运行（5.4.8+）（注意，不要用在生产环境！）

在后一种情况下，请执行以下操作：

	cd path/to/install
	php -S 0.0.0.0:8888 -t public public/index.php

然后，您可以访问我们的网站为<http://localhost:8888> - 这将弹出一个欢迎页面，并参观了仪表板，以创造并检查您的API的能力。


> ### 使用PHP内置Web服务器的注意:
> PHP的内置Web服务器开始不支持`PATCH` HTTP方法，直到5.4.8。由于管理API制作使用该HTTP方法，必须使用内置的Web服务器时使用的版本>=5.4.8。

> ### 关于Opcache的注意： ###
> **运行管理时，禁用所有opcode缓存！**
>
> 在管理后台使用opcode缓存将会无法正常运行，例如开启了APC或Opcache。Apigility不使用数据库来存储配置，它使用PHP配置文件。Opcode缓存将在这些文件第一次加载时缓存，从而导致不一致。通常会导致admin所在的API和代码变得不可用的状态。
> admin是一个开发工具，在开发环境中使用。因此，无论如何你应该禁用opcode缓存。
> 当你准备好部署你的API用于生产环境，你可以禁用开发环境模式，然后禁用管理界面，安全地重新运行opcode缓存。建议这样做用于生产环境，opcode缓存会提供极大的性能优势。
