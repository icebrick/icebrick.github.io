---
title: 网站中使用markdown并实现代码高亮
key: 20180415
tags: TeXt
---

在django管理后台admin中集成mardown模块
<!--more-->

### 需求：

1.  个人博客网站，在管理后台admin增加文章时，使用markdwon编辑器写文章；
2.  在博客显示页面将markdwon文章渲染成带格式的html；
3. 文章内容中实现代码高亮；
4. 修改编辑器的默认样式。

### 环境：

+ django==1.11.6
+ django-markdownx==2.0.22

---------------------------------------

之前在网站后台写文章使用的是百度的`UEditor`编辑器，该编辑器没有官方的django版本，我用的是第三方作者开发的DjangoUeditor。这个编辑器功能很多，但是使用体验不太好，代码高亮渲染后样式有点奇怪，而且我也用不了那么多功能，有点冗余。于是想换用轻量级markdown编辑器，也能轻松实现自己想要的文章格式。

在<a href="https://djangopackages.org/grids/g/markdown/" target="_blank">djangopackages</a>这个网站上比较了几款markdown应用，选择了<a href="https://github.com/neutronX/django-markdownx" target="_blank">django-markdownx</a>。

以下步骤是我个人在集成中使用的方法，参照自官方文档，因为有一些需求在文档中没找到明确的说明，于是部分结合了stackoverflow上的回答，这里做个总结备忘。

### 1. 在admin页面集成markdwon编辑器

这一步基本按照官方文档一步步进行就能完成。
首先安装django-markdwonx：

	pip install django-markdownx

在django project的`settings.py`中INSTALLED_APPS设置增加django-markdownx

	# settings.py
	INSTALLED_APPS = (
	    # [...]
	     'markdownx',
	)

在根`url`的设置中增加markdown的路径

	# urls.py
	urlpatterns = [
	    [...]
	    url(r'^markdownx/', include('markdownx.urls')),
	]

博客文章的模型定义如下，将`content`的`field`类型换成`MarkdownxField()`

	# models.py
	from markdownx.models import MarkdownxField
	class Article(models.Model):
	    title = models.CharField(max_length=100)
	    content = MarkdownxField()

在admin中用`MarkdownxModelAdmin`注册我们的模型

	# admin.py
	from django.contrib import admin
	from markdownx.admin import MarkdownxModelAdmin
	from .models import Article

	admin.site.register(Article, MarkdownxModelAdmin)

当再次打开网站的admin页面时，content这个字段已经被渲染成markdown编辑器，上边的文本输入框输入用markdown语法编写的内容，下边即时显示带格式文本。
它会自动处理好图片的上传，只要我们事先设置好`MEDIA_ROOT`和`MEDIA_URL`。

![截图.png](https://note.youdao.com/yws/res/12842/WEBRESOURCEadc315dd842815d2284ce9d906badf8c)

### 将markdown文章渲染成带格式的html

这个功能在django-markdownx包中已经实现，但是在它的文档中并没有明确提到，我google了一下，在stackoverflow上找到解决方法，其实很简单，只是因为我第一次使用markdown，没有弄清原理。

使用第一步编辑的文章保存到数据库里的内容是原始的markdown语法，如：

	标题
	=====
	副标题
	------

	![](/media/markdownx/ba1ccdee-ee19-4550-b17c-98443c09cc97.jpg)

如果我们直接把这些字符发送到前端模板，文章只会原样显示上面的字符，将不会显示格式，也不会显示图片。

而`markdownx.utils`中的`markdownnify`函数负责将原始markdown内容渲染成带格式html。所有需要在Article这个模型里增加一个属性：

	from markdownx.utils import markdownify
	from markdownx.models import MarkdownxField
	class Article(models.Model):
	    title = models.CharField(max_length=100)
	    content = MarkdownxField()
	    @property
	    def formatted_markdown(self):
	        return markdownify(self.content)

然后在模板中显示文章内容的地方使用这个属性：

	<p>{{ article.formatted_markdown | safe }}</p>

这样文章内容就能正常按格式显示在浏览器中。

### 文章内容中代码的高亮

这需要用到markdown的extentions中的模块，markdown自带<a href="https://python-markdown.github.io/extensions/code_hilite/" target="_blank">CodeHilite</a>模块。

在settings.py中增加`MARKDOWNX_MARKDOWN_EXTENSIONS`参数，以使用codehilite功能，并且设置使用行号。

	MARKDOWNX_MARKDOWN_EXTENSIONS = ['markdown.extensions.codehilite']
	MARKDOWNX_MARKDOWN_EXTENSION_CONFIGS = {
	    'markdown.extensions.codehilite': {
	        'linenums': True
	    }
	}

设置完之后需要生成用于代码高亮css文件，在命令行中运行：

	pygmentize -S default -f html -a .codehilite > styles.css

将生成styles.css文件，将文件拷贝到你的项目文件里，并在需要代码高亮的模板的头文件中引用该css。

刷新页面后会发现你文章中的代码块已经可以高亮了。

### 修改编辑器的默认样式

默认情况下，配置好的编辑器如图所示：

![](/media/markdownx/9ca74022-28a4-4c5c-916c-588aecd7c986.png)

编辑器和预览上下排列，当书写内容多了以后非常不方便查看，这里将其设置为左右排列。

在项目文件夹下面`templates`文件夹下创建`markdownx`文件夹，并将`markdownx/templates/markdownx/widget2.html`复制到新建的markdownx文件夹（对于django<1.11，复制widget.html文件）。`widget2.html`文件中原来的代码为：

	<div class="markdownx">
	    {% include 'django/forms/widgets/textarea.html' %}
	    <div class="markdownx-preview"></div>
	</div>

将其修改为：


```
	<br><br>
	<div class="container-fluid">
	<div class="row markdownx">
	    <div class="col-lg-6">
	    {% include 'django/forms/widgets/textarea.html' %}
	    </div>
	    <div class="col-lg-6">
	    <div class="markdownx-preview"></div>
	    </div>
	</div>
	</div>
```

因为修改后的html中用到了css col-lg-6，所以该页面所在的模板中应该要包含`bootstrap`的css文件。
<table><tr><td bgcolor=orange>PS: 我用的是版本3.3.6，想用cdn引用最新版本的bootstrap时发现有点问题，暂时用这个版本</td></tr></table>

为了让`django`能够正确的找到我们修改过的`widget2.html`，在`settings.py`中增加如下设置：

	# settings.py
	INSTALLED_APPS = (
	    # [...]
	     'django.forms',
	)
	FORM_RENDERER = 'django.forms.renderers.TemplatesSetting'

重启服务后，编辑器和预览栏就横排显示了
