---
layout: post
title:  "Class based view"
date:   2014-05-02
categories: documentation
---


.. versionadded:: 1.3

.. note::
    在Django1.3之前，通用视图是以函数的方式来实现的。基于函数的实现已经不
    建议使用，建议使用这里介绍的基于类的实现方式。

    要了解更多上面说的通过视图的实现可以查看
    :doc:`topic guide </topics/generic-views>` 和
    :doc:`detailed reference </ref/generic-views>`.

编写Web应用可能是单调的，因为你需要不断的重复某一种模式。Django尝试从model和
template层移除一些单调的情况，但是Web开发者依然会在view（视图）层经历这种厌烦。

Django的 *通用视图* 被开发用来消除这一痛苦。它们采用某些常见的习语和在开发过
程中发现的模式然后把它们抽象出来，以便你能够写更少的代码快速的实现基础的视图。

我们能够识别一些基础的任务，比如展示对象的列表，以及编写代码来展示任何对象的
列表。此外，有问题的模型可以作为一个额外的参数传递到URLconf中。

Django通过通用视图来完成下面一些功能：

* 执行基础的"简单的"任务：重定向到一个不同的页面以及渲染一个给定
  的模板。

* 为单一的对象展示列表和一个详细页面。如果我们创建一个应用来管理会议，那么
  一个 ``TalkListView`` (讨论列表视图)和一个 ``RegisteredUserListView`` （
  注册用户列表视图）就是列表视图的一个例子。一个单独的讨论信息页面就是我们称
  之为 "详细" 视图的例子。

* 在 年/月/日的归档页面中，有关联的详细页面以及"最近更新"页面中提供基于日期
  的对象。 `The Django Weblog <https://www.djangoproject.com/weblog/>`_ 中的
  年，月，日归档就是基于此来构建的，就像典型的新闻归档一样。

* 允许用户创建，更新和删除对象 -- 以授权或者无需授权的方式。

总的来说，这些视图提供了一些简单的接口来完成开发者遇到的大多数的常见任务。


Simple usage 简单使用
====================================

基于类的通用视图（以及任何继承了Django提供的基础类的基于类的视图）都能够
以下面两种方式被配置：子类化，或者直接通过URLconf来传递参数。

当你子类化一个类视图时，你可以重写一些属性（比如 ``template_name`` ）或者
一些方法（比如 ``get_context_data`` ）在你的子类中来提供一些新的值或者方
法。考虑一下，比如，一个仅仅需要展示一个模板的视图， ``about.html`` 。
Django有一个通用视图来完成这个功能 - :class:`~django.view.generic.base.TemplateView`
- 因此你可以子类化它，然后重写模板的名称::

    # some_app/views.py
    from django.views.generic import TemplateView

    class AboutView(TemplateView):
        template_name = "about.html"

这时，你只需要添加这个新的视图到你的URLconf配置中。因为类视图本身是一个类，我
们把URL指向 ``as_view`` 这个类方法来替代类本身，这是类视图的入口点::

    # urls.py
    from django.conf.urls import patterns, url, include
    from some_app.views import AboutView

    urlpatterns = patterns('',
        (r'^about/', AboutView.as_view()),
    )

作为一个选择，如果你仅仅修改类视图中少量简单的属性，你可以直接传递新的属性
到类本身调用 ``as_view`` 方法中::

    from django.conf.urls import patterns, url, include
    from django.views.generic import TemplateView

    urlpatterns = patterns('',
        (r'^about/', TemplateView.as_view(template_name="about.html")),
    )

一个类似的重写模式可以用在 :class:`~django.views.generic.base.RedirectView`
的 ``url`` 属性上，这是另外一个简单的通用视图。


Generic views of objects 对象的通用视图
================================================

:class:`~django.views.generic.base.TemplateView` 确实很有用，但是当你需要
呈现你数据库中的内容时Django的通用视图才真的会脱颖而出。因为这是如此常见
的任务，Django提供了一大把内置的通用视图，使生成对象的展示列表和详细视图
的变得极其容易。

让我们来看一下这些通用视图中的："对象列表"视图。我们将使用下面的模型(models)::

    # models.py
    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

        class Meta:
            ordering = ["-name"]

        def __unicode__(self):
            return self.name

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField('Author')
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

为了给所有的Publisher建立一个列表页，我们将按照这样的方式来配置URLconf::

    from django.conf.urls import patterns, url, include
    from django.views.generic import ListView
    from books.models import Publisher

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            model=Publisher,
        )),
    )

这就是我们要写的所有Python代码。无论如何，我们仍然需要写一个模板。我们可以明确
的告诉视图(view)来使用哪个模板，通过在给as_view传递参数时包含一个
``template_name`` 的关键字参数，但是在缺乏template参数时Django会推断名字就是对
象的名字。在这种情况下，推断出来的模板将是这样的 ``"books/publisher_list.html"`` 
-- 其中的 "books" 部分来自模型(model)所属的app的名字，而 "publisher" 部分仅仅
是模型名字的小写字母。

.. note::
    因此，当（比如说） :class:`django.template.loaders.app_directories.Loader`
    这个模板加载器在添加到 :setting:`TEMPLATE_LOADERS`  中时，这个模板的路径将是::

        /path/to/project/books/templates/books/publisher_list.html

.. highlightlang:: html+django

这个模板将会依据于一个上下文(context)来渲染，这个context包含一个名为 ``object_list``
包含所有publisher对象的变量。一个非常简单的模板可能看起来就像下面这个::

    {% extends "base.html" %}

    {% block content %}
        <h2>Publishers</h2>
        <ul>
            {% for publisher in object_list %}
                <li>{{ publisher.name }}</li>
            {% endfor %}
        </ul>
    {% endblock %}

这确实就是全部代码了。所有通用视图中酷的特性来自于修改被传递到通用视图中的"信息"
字典。这篇文档 :doc:`generic views reference</ref/class-based-views>` 详细的
介绍了通用视图以及它的选项;本篇文档剩余的部分将会考虑你可以自定义以及扩展通用
视图的常见方法。


Extending generic views 扩展通用视图
==============================================

.. highlightlang:: python

这是毫无疑问的，使用通用视图可以极大的提高开发速度。然而，在大多数工程中，
总会遇到这样的时刻，通用视图无法满足需求。的确，大多数来自Django开发新手
的问题是如何能使得通用视图的使用范围更广。

这是通用视图在1.3发布中被重新设计的原因之一 - 之前，它们仅仅是一些函数视图加上
一列令人疑惑的选项;现在，比起传递大量的配置到URLconf中，更推荐的扩展通用视图的
方法是子类化它们，并且重写它们的属性或者方法。


Making "friendly" template contexts 编写"友好的"模板上下文
----------------------------------------------------------------------

你可能已经注意到了，我们在publisher列表的例子中把所有的publisher对象
放到 ``object_list`` 变量中。虽然这能正常工作，但这对模板作者并不是
"友好的"。他们只需要知道在这里要处理publishers就行了。

因此，如果你在处理一个模型(model)对象，这对你来说已经足够了。当你处理
一个object或者queryset时，Django能够使用你定义对象显示用的verbose name
（或者plural verbose name，对于对象列表）来填充上下文(context)。这被
提供添加到默认的 ``object_list`` 实体中，但是包含完全相同的数据。

如果verbase name(或者plural verbose name) 仍然不能很好的符合要求，你
可以手动的设置上下文(context)变量的名字。通用视图中的这个属性 :
``context_object_name`` 指定上下文(context)变量要使用的名字。在这个例子
中我们在URLconf中重写了它，因为这只是简单的修改:

.. parsed-literal::

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            model=Publisher,
            **context_object_name="publisher_list",**
        )),
    )

提供一个有用的 ``context_object_name`` 总是个好主意。和你一起工作的设计
模板的同事会感谢你的。


Adding extra context 添加额外的上下文
----------------------------------------

经常的你需要提供一些通用视图没有提供的额外的信息。比如，考虑到每个publisher
详细页面上的图书列表的展示。 :class:`~django.views.generic.detail.DetailView`
通用视图提供了一个publisher对象给context，但是看起来在模板中没
有获取附加信息的途径。

然而，是有方法的;你可以子类化 :class:`~django.views.generic.detail.DetailView`
然后提供你自己的 ``get_context_data`` 方法的实现。 
:class:`~django.views.generic.detail.DetailView` 中默认的实现只是简单的
给模板添加了要展示的对象，但是你这可以这么重写来展示更多信息::

    from django.views.generic import DetailView
    from books.models import Publisher, Book

    class PublisherDetailView(DetailView):

        context_object_name = "publisher"
        model = Publisher

        def get_context_data(self, **kwargs):
            # Call the base implementation first to get a context
            context = super(PublisherDetailView, self).get_context_data(**kwargs)
            # Add in a QuerySet of all the books
            context['book_list'] = Book.objects.all()
            return context


Viewing subsets of objects 查看对象的子集合
----------------------------------------------------

现在让我们来近距离查看下我们一直在用的 ``model`` 参数。这个 ``model``
参数指定了视图(view)在哪个数据库模型之上进行操作，这适用于所有的需要
操作一个单独的对象或者一个对象集合的通用视图。然而，这个 ``model`` 
参数并不是唯一能够指明视图(view)要基于哪个对象进行操作的方法 -- 
你同样可以使用 ``queryset`` 参数来指定一个对象列表::

    from django.views.generic import DetailView
    from books.models import Publisher, Book

    class PublisherDetailView(DetailView):

        context_object_name = "publisher"
        queryset = Publisher.objects.all()

指明 ``model = Publisher`` 等价于快速声明的 ``queryset = Publisher.objects.all()``
。然而，通过使用 ``queryset`` 来定义一个过滤的对象列表，你可以更加详细
的了解哪些对象将会被显示的视图(view)中(看 :doc:`/topics/db/queries`
来获取更多关于 :class:`QuerySet` 对象的更对信息，以及看
:doc:`class-based views reference </ref/class-based-views>` 来获取全部
细节)。

选择一个简单的例子，我们可能想要对books列表按照出版日期进行排序，把
最近的放到前面::

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            queryset=Publisher.objects.all(),
            context_object_name="publisher_list",
        )),
        (r'^books/$', ListView.as_view(
            queryset=Book.objects.order_by("-publication_date"),
            context_object_name="book_list",
        )),
    )


这是个非常简单的列子，但是它很好的诠释了处理思路。当然，你通常想做的不仅仅只是
对对象列表进行排序。如果你想要展现某个publisher的所有图书(book)列表，你可以使用
同样的手法（这次，通过子类化的而不是传递参数到URLconf的方式来说明）::

    from django.views.generic import ListView
    from books.models import Book

    class AcmeBookListView(ListView):

        context_object_name = "book_list"
        queryset = Book.objects.filter(publisher__name="Acme Publishing")
        template_name = "books/acme_list.html"

注意，和经过过滤之后的 ``queryset`` 一起定义的还有我们自定义的模板名称。
如果我们不这么做，通过视图会使用和 "vanilla" 对象列表名称一样的模板，这可
能不是我们想要的。

另外需要注意，这并不是非常优雅的方法来处理特定publisher的图书(book)。如果我们 
要创建另外一个publisher页面，我们需要添加另外几行代码到URLconf中，并且再多几个
publisher就会觉得这么做不合理。我们会在下一个章节处理这个问题。

.. note::

    如果你在访问 ``/books/acme/`` 是出现404错误，检查确保你确实有一个名字为
    'ACME Publishing'的Publisher。通用视图在这种情况下提供一个 ``allow_empty``
    的参数。看 :doc:`class-based-views reference</ref/class-based-views>` 
    获取更多信息。


Dynamic filtering 动态过滤
----------------------------------

另一个普遍的需求是在给定的列表页面中根据URL中的关键字来过滤对象。前面我们把出版
商的名字硬编码到URLconf中，但是如果我们想要编写一个视图来展示任何publisher的所有
图书，怎么处理？

很方便， ``ListView`` 有一个 
:meth:`~django.views.generic.detail.ListView.get_queryset` 方法来供我们重写。
在之前，它只是返回一个 ``queryset`` 属性值，但是现在我们可以添加更多的逻辑。

让这种方式能够工作的关键点在于当类视图被调用时，各种有用的对象被存储在 ``self`` 上;
同request(``self.request``)一样，其中包含了从URLconf中获取到的位置参数
(``self.args``)和基于名字的参数(``self.kwargs``)(关键字参数，译者注)。

这里，我们有一个URLconf定义了一组供捕获的参数::

    from books.views import PublisherBookListView

    urlpatterns = patterns('',
        (r'^books/(\w+)/$', PublisherBookListView.as_view()),
    )

下一个，我们定义了 ``PublisherBookListView`` 视图::

    from django.shortcuts import get_object_or_404
    from django.views.generic import ListView
    from books.models import Book, Publisher

    class PublisherBookListView(ListView):

        context_object_name = "book_list"
        template_name = "books/books_by_publisher.html"

        def get_queryset(self):
            publisher = get_object_or_404(Publisher, name__iexact=self.args[0])
            return Book.objects.filter(publisher=publisher)

如你所见，可以非常容易的在queryset区域添加更多的逻辑;如果我们想的话，我们可以
使用 ``self.request.user`` 来过滤当前用户，或者添加其他更复杂的逻辑。

同时我们可以把publisher添加到context中，这样我们就可以在模板中使用它::

    class PublisherBookListView(ListView):

        context_object_name = "book_list"
        template_name = "books/books_by_publisher.html"

        def get_queryset(self):
            self.publisher = get_object_or_404(Publisher, name__iexact=self.args[0])
            return Book.objects.filter(publisher=self.publisher)

        def get_context_data(self, **kwargs):
            # Call the base implementation first to get a context
            context = super(PublisherBookListView, self).get_context_data(**kwargs)
            # Add in the publisher
            context['publisher'] = self.publisher
            return context

Performing extra work 执行额外的工作
------------------------------------------

我们需要考虑的最后的共同模式在调用通用视图之前或者之后会引起额外的开销。
The last common pattern we'll look at involves doing some extra work before
or after calling the generic view.

想象一下，在我们的 ``Author`` 对象上有一个 ``last_accessed`` 字段，这个字段用来
跟踪某人最后一次查看了这个作者的时间。

    # models.py

    class Author(models.Model):
        salutation = models.CharField(max_length=10)
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField()
        headshot = models.ImageField(upload_to='/tmp')
        last_accessed = models.DateTimeField()

通用的 ``DetailView`` 类，当然不知道关于这个字段的事情，但再一次我们可以很容易
编写一个自定义的视图来保持这个字段的更新。

首先，我们需要添加author详情页的代码配置到URLconf中，指向自定义的视图。

.. parsed-literal::

    from books.views import AuthorDetailView

    urlpatterns = patterns('',
        #...
        **(r'^authors/(?P<pk>\\d+)/$', AuthorDetailView.as_view()),**
    )

然后，编写我们新的视图 -- ``get_object`` 是用来获取对象的方法 -- 因此我们简单的
重写它并封装调用::

    import datetime
    from books.models import Author
    from django.views.generic import DetailView
    from django.shortcuts import get_object_or_404

    class AuthorDetailView(DetailView):

        queryset = Author.objects.all()

        def get_object(self):
            # Call the superclass
            object = super(AuthorDetailView, self).get_object()
            # Record the last accessed date
            object.last_accessed = datetime.datetime.now()
            object.save()
            # Return the object
            return object

.. note::

    这段代码并不能运行，除非你创建一个 ``books/author_detail.html``
    的模板。

.. note::

    这里的URLconf使用参数组的名字 ``pk`` - 这个名字是 ``DetailView`` 用来查找
    用来queryset的主键的值的默认名称。

    如果你想要修改它，你需要在你的 ``get()`` 中使用从 ``self.kwargs`` 中获取
    到的新的参数名称来调用 ```self.queryset`。

More than just HTML 不仅仅是HTML
--------------------------------------

到目前为止，我们关注在渲染模板到生成responses(响应)上。然而，
通用视图能够做的远不止这些。

每一个通用视图都是由一系列的mixin组成的，而每个mixin给整个视图贡献
了一小片代码。其中一些mixin -- 比如
:class:`~django.views.generic.base.TemplateResponseMixin` -- 是为
使用一个模板渲染内容到html特殊设计的。无论如何，你都可以编写自己的
mixin来完成不同的渲染行为。

比如，一个简单的处理JSON的mixin可能看起来是这样的::

    from django import http
    from django.utils import simplejson as json

    class JSONResponseMixin(object):
        def render_to_response(self, context):
            "Returns a JSON response containing 'context' as payload"
            return self.get_json_response(self.convert_context_to_json(context))

        def get_json_response(self, content, **httpresponse_kwargs):
            "Construct an `HttpResponse` object."
            return http.HttpResponse(content,
                                     content_type='application/json',
                                     **httpresponse_kwargs)

        def convert_context_to_json(self, context):
            "Convert the context dictionary into a JSON object"
            # Note: This is *EXTREMELY* naive; in reality, you'll need
            # to do much more complex handling to ensure that arbitrary
            # objects -- such as Django model instances or querysets
            # -- can be serialized as JSON.
            return json.dumps(context)

这时，你就可以构建一个返回JSON数据的
:class:`~django.views.generic.detail.DetailView` 通过混合你的
:class:`JSONResponseMixin` 和
:class:`~django.views.generic.detail.BaseDetailView` -- (类
:class:`~django.views.generic.detail.DetailView` 在模板渲染行
为之前被混入其他行为)::

    class JSONDetailView(JSONResponseMixin, BaseDetailView):
        pass

这个视图这次可以像其它任何 :class:`~django.views.generic.detail.DetailView` 
类一样被部署，用完全相同的行为 -- 除了响应的格式之外。

如果你想要冒险，你甚至可以混入一个 :class:`~django.views.generic.detail.DetailView`
的子类，这个子类能够同时返回HTML和JSON内容，依赖于HTTP请求中的一些属性，
比如一个查询参数或者是一个HTTP头部。仅仅是混合
:class:`JSONResponseMixin` 和
:class:`~django.views.generic.detail.SingleObjectTemplateResponseMixin`,
然后重写方法 :func:`render_to_response()` 依靠用户请求中需要的响应类型
把实现延迟到适当的子类中::

    class HybridDetailView(JSONResponseMixin, SingleObjectTemplateResponseMixin, BaseDetailView):
        def render_to_response(self, context):
            # Look for a 'format=json' GET argument
            if self.request.GET.get('format','html') == 'json':
                return JSONResponseMixin.render_to_response(self, context)
            else:
                return SingleObjectTemplateResponseMixin.render_to_response(self, context)

因为Python解析方法重载的方式，局部的 ``render_to_response()`` 实现将会
覆盖由
:class:`JSONResponseMixin` 和
:class:`~django.views.generic.detail.SingleObjectTemplateResponseMixin`.
提供的版本。

Decorating class-based views 装饰类视图
========================================================

.. highlightlang:: python

对于类视图的扩展并不局限于使用mixin。你也可以使用装饰器。

Decorating in URLconf URLconf中的装饰器
------------------------------------------

最简单的装饰类视图的方式是装饰 :meth:`~django.views.generic.base.View.as_view` 
方法返回的结果。最容易装饰的地方是你配置你的视图的地方URLconf中::

    from django.contrib.auth.decorators import login_required, permission_required
    from django.views.generic import TemplateView

    from .views import VoteView

    urlpatterns = patterns('',
        (r'^about/', login_required(TemplateView.as_view(template_name="secret.html"))),
        (r'^vote/', permission_required('polls.can_vote')(VoteView.as_view())),
    )

这种方法适用于装饰每个实例的基础。如果你想要视图的每个实例都被装饰，
你需要采用别的方法。

.. _decorating-class-based-views:

Decorating the class 装饰类
----------------------------------------

要装饰类视图的每一个实例，你需要装饰类自身的定义。要实现这个目的，你要
应用这个装饰器到类的 :meth:`~django.views.generic.base.View.dispatch` 
方法上。

类方法和独立的函数并不完全一样，因此你不能仅仅应用函数的装饰器 --
你需要先把它转换为一个类方法的装饰器。 ``method_decorator`` 这个装饰器
能把一个函数装饰器转变为类方法装饰器，因此它就可以被用在实例方法上了。
比如::

    from django.contrib.auth.decorators import login_required
    from django.utils.decorators import method_decorator
    from django.views.generic import TemplateView

    class ProtectedView(TemplateView):
        template_name = 'secret.html'

        @method_decorator(login_required)
        def dispatch(self, *args, **kwargs):
            return super(ProtectedView, self).dispatch(*args, **kwargs)

在这个例子中， ``ProtectedView`` 的每个实例都将拥有登录保护的功能。

.. note::

    ``method_decorator`` 方法传递 ``*args`` 和 ``**kwargs`` 给这个类上的
    装饰器。如果你的方法不接受兼容参数集合，它会抛出 ``TypeError`` 异常。
