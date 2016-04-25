title: CSS基本样式
date: 2016-02-19 13:46:45
tags: html5
---

**CSS:(cascading style sheet)一般称为层叠样式表**
,一种用来为结构化文档（如HTML文档或XML应用）添加样式（字体、间距和颜色等）的计算机语言.

网页的读者和作者都可以使用CSS来决定文件的颜色、字体、排版等显示特性,比如Chrome浏览器的[Stylish](https://userstyles.org/)插件可以自定义网页的css文件,来修改网站样式。CSS最主要的目的是将文件的内容与显示分隔开来。

这有许多好处：

* 文件的可读性加强
* 文件的结构更加灵活
* 作者和读者可以自己决定文件的显示
* 文件的结构简化了
另外，在HTML中：

一个整个网站或其中一部分网页的显示信息被集中在一个地方，要改变它们很方便

不同的读者可以有不同的样式，比如有的读者需要字体比较大

HTML文件本身的范围变小了，它的结构简单了，它不需要包含显示的信息

CSS还可以控制其他参数，例如声音（假如浏览器有阅读功能的话）或给视障者用的感受装置。

### 几种引入方式

*  外部引入:

	在head中可以引入另外一个单独的css文件

	```
	<link rel="stylesheet" href="test.css">
	```
	

	>  * 一个css文件可以控制多个html页面
> 
>  * 方便改版和维护 
> 
>  * 有效利用浏览器的缓存,实现加载更快
> 
>  * 相对于整站来说,代码更少,更加方便于分工合作
> 
>  * 相对于单页面来说,有垃圾代码,而且link标签中的href会增加服务器的压力
> 
>  * 整站访问量不是特别大的网页中

  

* 头部引用:放在头部的style中

	```
	<style type="text/css">
		div{
			color: #999;
		}
		</style>
	```


	>  * 速度快(相对于网络层面),没有请求压力
> 
>  * 相对外部引入来说代码更少
> 
>  * 不宜改版和维护
> 
>  * 代码很乱,不利于前后台的沟通
> 
>  * 使用场景:高访问量的网页(首页,活动页)


* 标签内引用

	```
	<div style="color: blue; ">我是div</div>
	```
	>  1.优先级最高
> 
>  2.多余代码多
> 
>  3.不利于维护
> 
>  使用场景:跟js和后台生成


 优先级:外部-头部-标签内

 范围越小,优先级越高

---


### 选择器

* 标签名选择器

	```
	div{
		color: red;
	}
	```

* class选择器

	```
	<a href="###" class="red">超链接</a>
	```
	```
	.red{
		color:red;
	}
	```

* id选择器

	```
	<a href="###" id="red">超链接</a>
	```
	```
	#red{
	 	color:red;
	}
	```
* 组合选择器

	1. div标签下的所有a标签的样式,包括嵌套后的a标签(不推荐使* 用)

  		```
		<div>
			  <a href="###">a标签</a>
		</div>
		```
		```
		div a{
			color:red;
			}
		```

	2. div的下一级a标签有效(仅CSS3支持) 

		```
		<div>
		 	<a href="###">###</a>
		</div>
		```
		```
		div>a{
			color:red;
		 }
		```

	3. div标签下a标签class为green的样式

		```
		<div>
			<a href="###" class="green">###</a>
		</div>
		```
		```
		div a.green{
			color:green;
		}
		```
	4. 多个标签,class,id使用同一种样式

		```
		<p>这是p</p>
		<div>这是div</div>
		<a href="###" class="green">我是a1</a>
		<a href="###" class="red" id="green">我是a2</a>
		<a href="###">我是a3</a>
		<a href="###" class="red">我是a4</a>
		<a href="###">我是a5</a>
		```
		```
		div,p,.red,#green{
			color:red;
		}
		```

---

### 常用样式

* 文字样式

	```
	p{
		font-family:"宋体";
		font-size:20px;
		font-style:normal;
	  	/*
  		normal  - 文本正常显示
	  	italic - 文本斜体显示
  		oblique - 文本倾斜显示;
	  	*/
  		font-weight:bold;
	}
	```

* 文本样式

	```
	p{
		width: 200px;
		height: 200px;
		/*边框*/
		border: 1px solid red;
		/*行高*/
		line-height: 200px;
		/*文本修饰*/
		/*下划线*/
		text-decoration: underline;
		/*删除线*/
		text-decoration: line-through;
		/*文本缩进*/
		/*浏览器默认一个字16px*/
		text-indent: 32px;
	}
	a{
	  /*取消a标签的下划线,其实是取消所有样式设置*/
		text-decoration: none;
	}
```