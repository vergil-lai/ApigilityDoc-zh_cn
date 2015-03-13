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

集合在HAL的字面上只是嵌入资源的数组。一个典型的集合将包括`_link`关系链接，但是还有分页链接-`first`，`last`，`next`，和`prev`是标准的关系。通常的API也会显示资源有多少被传递在当前载体的总数量，以及有关的集合潜在的其他元数据。

    {
        "_links": {
            "self": {
                "href": "http://example.org/api/user?page=3"
            },
            "first": {
                "href": "http://example.org/api/user"
            },
            "prev": {
                "href": "http://example.org/api/user?page=2"
            },
            "next": {
                "href": "http://example.org/api/user?page=4"
            },
            "last": {
                "href": "http://example.org/api/user?page=133"
            }
        }
        "count": 3,
        "total": 498,
        "_embedded": {
            "users": [
                {
                    "_links": {
                        "self": {
                            "href": "http://example.org/api/user/mwop"
                        }
                    },
                    "id": "mwop",
                    "name": "Matthew Weier O'Phinney"
                },
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
            ]
        }
    }

集合的各种关系链接可以轻松地遍历这个API的完整列表的资源集合。你可以很容易地确定你是在哪一页，下一个页面应该是什么（并且如果你在最后一页）。

集合中的每一项资源，包含一个链接到本身，所以你可以得到完整的资源，但也知道它的规范位置。通常，你可能不完全嵌入资源集合，只是相关的部分做一个快速的列表。因此，你获取的全部细节后，如果需要允许有链接到单个资源。

与HAL交互
---------

与HAL交互通常是非常直接的：

* 发出请求，`Accept`标头值使用`application/json`或者`application/hal+json`（虽然后者不是必需的）。
* 如果`POST`，`PUT`，`PATCH`，或者`DELETE`一个资源，你通常会使用一个`Content-Type`标头是`application/json`，或一些特别的提供商定义为你的API定义的Mediatype。这个MediaType将用于描述你的资源的特定结构，而没有任何HAL的`_links`。任何`_embedded`资源通常被描述为资源的属性，并指向嵌入资源的MediaType。
* 该API将响应一个MediaType为`application/hal+json`的。

当创建或更新一个资源（或集合），你会提交对象，没有关系的链接；该API会负责分配链接。如果我们考虑上面的嵌入式资源的例子，我将创建他是这样的：

    POST /api/user HTTP/1.1
    Accept: application/json
    Content-Type: application/vnd.example.user+json
    {
        "id": "matthew",
        "name": "Matthew Weier O'Phinney",
        "contacts": [
            {
                "id": "mac_nibblet"
            },
            {
                "id": "spiffyjr"
            }
        ],
        "website": {
            "id": "mwop"
        }
    }

响应应该像这样:

    HTTP/1.1 201 Created
    Content-Type: application/hal+json
    Location: http://example.org/api/user/matthew
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

`PUT`和`PATCH`的操作相同。