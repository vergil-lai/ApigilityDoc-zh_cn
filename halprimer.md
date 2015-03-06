超媒体应用程序语言（HAL）
========================

HAL是“Hypermedia Application Language（超媒体应用程序语言）”的缩写，是一个[开放规范描述的通用结构RESTful资源](http://stateless.co/hal_specification.html)。它提出的结构容易实现[Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)的Level3，通过确保每个资源包关系链接，一个标准，
识别结构存在嵌入其它资源。

从本质上讲，良好的REST风格的API应该：

* 公开的资源
* 通过HTTP，使用HTTP动词来操作它们
* 并提供规范自己的链接,链接到其他相关资源。


超媒体类型
----------
HAL提出了两种超媒体资源类型，XML和JSON。通常，API返回的类型是唯一相关的资源，因为关系链接通常不提交创建，更新或删除的资源。

HAL定义了JSON API的通用媒体类型是`application/hal+json`。

资源
----
对于JSON资源，最低限度你必须做的就是提供一个`_links`属性包含一个`self`的关系链接。例子：

	{
    	"_links": {
        	"self": {
            	"href": "http://example.org/api/user/matthew"
        	}
    	}
    	"id": "matthew",
    	"name": "Matthew Weier O'Phinney"
	}

如果你是包括其他资源嵌入在你所代表的资源，你将提供一个`_embedded`的属性，包含命名资源。每个资源将会被构造为HAL资源，并包含至少一个`_link`属性和`self`关系链接。

    {
        "_links": {
            "self": {
                "href": "http://example.org/api/user/matthew"
            }
        }
        "id": "matthew",
        "name": "Matthew Weier O'Phinney",
        "_embedded": {
            "contacts": [
                {
                    "_links": {
                        "self": {
                            "href": "http://example.org/api/user/mac_nibblet"
                        }
                    },
                    "id": "mac_nibblet",
                    "name": "Antoine Hedgecock"
                },
                {
                    "_links": {
                        "self": {
                            "href": "http://example.org/api/user/spiffyjr"
                        }
                    },
                    "id": "spiffyjr",
                    "name": "Kyle Spraggs"
                }
            ],
            "website": {
                "_links": {
                    "self": {
                        "href": "http://example.org/api/locations/mwop"
                    }
                },
                "id": "mwop",
                "url": "http://www.mwop.net"
            },
        }
    }

需要注意的是，在`_embedded`列表中每个元素可以是一个资源或者资源的数组。这需要我们下一个话题：集合。

集合
----

