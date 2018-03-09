---
layout:     post
title:      Admin和单例模式
subtitle:   Singleton Pattern
date:       2018-3-9
author:		youyue
header-img: 
catalog: true
tags:
    - Django
---
## admin的功能定制
### 两种定制方式:
```
方式一：
    class UserConfig(admin.ModelAdmin):
        list_display = ('user', 'pwd',)
 
    admin.site.register(UserInfo, UserConfig) # 第一个参数可以是列表
     
 
方式二：
    @admin.register(UserInfo)                # 第一个参数可以是列表
    class UserConfig(admin.ModelAdmin):
        list_display = ('user', 'pwd',)
```
		
### 常用定制功能,均添加在定制类UserConfig下:
```
1. list_display，列表时，定制显示的列。
2. list_display_links，列表时，定制列可以点击跳转。(a标签效果,与部分功能不能同时使用)
3. list_filter，列表时，定制右侧快速筛选。
4. list_select_related，列表时，连表查询是否自动select_related
5. list_editable，列表时，可以编辑的列 
6. search_fields，列表时，模糊搜索的功能
7. date_hierarchy，列表时，对Date和DateTime类型进行搜索
8. inlines，详细页面，如果有其他表和当前表做FK，那么详细页面可以进行动态增加和删除
9. action，列表时，定制action中的操作
	@admin.register(UserInfo)  # 装饰需要定制的表
	class UserAdmin(admin.ModelAdmin):
	 
		# 定制Action行为具体方法
		def func(self, request, queryset):
			print(self, request, queryset)
			queryset.update(email="iyouyue@qq.com")		# 点action时执行的代码
	 
		func.short_description = "action自定义显示的文字"
		actions = [func, ]
	 
		# Action选项都是在页面上方显示
		actions_on_top = True
		# Action选项都是在页面下方显示
		actions_on_bottom = False
	 
		# 是否显示选择个数
		actions_selection_counter = True

10.定制HTML模板
	add_form_template = None
	change_form_template = None
	change_list_template = None
	delete_confirmation_template = None
	delete_selected_confirmation_template = None
	object_history_template = None
11.raw_id_fields，详细页面，针对FK和M2M字段变成以Input框形式
12.fields，详细页面时，显示字段的字段
13.exclude，详细页面时，排除的字段
14.readonly_fields，详细页面时，只读字段
15.fieldsets，详细页面时，使用fieldsets标签对数据进行分割显示
	@admin.register(models.UserInfo)
	class UserAdmin(admin.ModelAdmin):
		fieldsets = (
			('基本数据', {
				'fields': ('user', 'pwd', 'ctime',)
			}),
			('其他', {
				'classes': ('collapse', 'wide', 'extrapretty'),  # 'collapse','wide', 'extrapretty'
				'fields': ('user', 'pwd'),
			}),
		)
16.filter_vertical，详细页面时，M2M显示时，数据移动选择（方向：上下和左右）
17.ordering，列表时，数据排序规则
18.radio_fields，详细页面时，使用radio显示选项（FK默认使用select）
19.form = ModelForm，用于定制用户请求时候表单验证
20.empty_value_display = "列数据为空时，显示默认值"
```
## 单例模式
单例模式（Singleton Pattern）是一种常用的软件设计模式，该模式的主要目的是**确保某一个类只有一个实例存在**。当你希望在整个系统中，某个类只能出现一个实例时，单例对象就能派上用场。	
比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象。

在 Python 中，我们可以用多种方法来实现单例模式：	
- 使用模块	
- 使用 __new__	
- 使用装饰器（decorator）			
- 使用元类（metaclass）	
### (1)使用 __new__		
```
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kw):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)  
        return cls._instance  

class MyClass(Singleton):  
    a = 1
```
在上面的代码中，我们将类的实例和一个类变量 _instance 关联起来，如果 cls._instance 为 None 则创建实例，否则直接返回 cls._instance。

执行情况如下：
```
>>> one = MyClass()
>>> two = MyClass()
>>> one == two
True
>>> one is two
True
>>> id(one), id(two)
(4303862608, 4303862608)
```
### (2)使用模块
其实，Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：
```
class My_Singleton(object):
    def foo(self):
        pass
 
my_singleton = My_Singleton()
```
将上面的代码保存在文件 mysingleton.py 中，然后这样使用：
```
from mysingleton import my_singleton
 
my_singleton.foo()
```

## admin执行流程
```
	<1> 循环加载执行所有已经注册的app中的admin.py文件
	def autodiscover():
		autodiscover_modules('admin', register_to=site)
	<2>执行代码
	＃admin.py

	class BookAdmin(admin.ModelAdmin):
		list_display = ("title",'publishDate', 'price')

	admin.site.register(Book, BookAdmin) 
	admin.site.register(Publish)
	<3>admin.site 
	路径:\django\contrib\admin\sites.py
	class AdminSite(object):
		...
		...
	# This global object represents the default admin site, for the common case.
	# You can instantiate AdminSite in your own code to create a custom admin site.
	site = AdminSite()

	<4> 执行register方法
	admin.site.register(Book, BookAdmin) 
	admin.site.register(Publish)
	class ModelAdmin(BaseModelAdmin):pass

	def register(self, model_or_iterable, admin_class=None, **options):
		if not admin_class:
				admin_class = ModelAdmin
		# Instantiate the admin class to save in the registry
		self._registry[model] = admin_class(model, self)

	到这里，注册结束！

	<5>admin的URL配置
	class AdminSite(object):
		
		 def get_urls(self):
			from django.conf.urls import url, include
		  
			urlpatterns = []

			# Add in each model's views, and create a list of valid URLS for the
			# app_index
			valid_app_labels = []
			for model, model_admin in self._registry.items():
				urlpatterns += [
					url(r'^%s/%s/' % (model._meta.app_label, model._meta.model_name), include(model_admin.urls)),
				]
				if model._meta.app_label not in valid_app_labels:
					valid_app_labels.append(model._meta.app_label)

		  
			return urlpatterns

		@property
		def urls(self):
			return self.get_urls(), 'admin', self.name
	<6> url()方法的扩展应用: 略
```