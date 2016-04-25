title: CSS3增加的一些选择器
date: 2016-02-29 11:49:16
tags: html5
---
### CSS3选择器

在CCS3中增加了一些组合选择器,方便更多条件的查找

* 子元素选择器

	```
	div>p{
		background-color: yellow;
	}
	```
表示div的下一级元素为p标签时p标签的样式.

* 相邻兄弟选择器(Adjacent sibling selector)

	```
	h1+p{
		color: yellow;
	}
	```
	标签样式如下,对相邻元素的后一个元素有效.
	注意两点:
	* 注意是**相邻**,所以中间如果有其他元素这个样式就无效了.
	* 相邻元素的后**一个**元素,所以如果之后再有p元素也不满足这个样式的条件


	```
	<h1>这是另一个h1</h1>
	<p>这是另一个p</p>
	```

* 同级元素通用选择器

	```
	h1~div{
		color: green;
	}
	```
	匹配任何在h1之后的同级div元素.
	和上一个标签作对比:

	* 没有相邻的条件约束,所以只要是h1之后的**同级**标签是div就有效
	* 没有后一个条件约束,h1之后所有的同级div标签都有效
	
* 属性选择器

	```
	*[title]{
		color: red;
		border: 5px solid red;
	}
	```
	匹配所有带有title的元素.可以多个属性查询,比如:

	```
	a[href][title]{
		color: red;
	}
	```

	还可以直接匹配属性的值,比如:

	```
	a[href="###"]{
		color: red;
	}
	```
	还可以使用正则表达式匹配,比如:

	```
	a[href^="#"]{   /*以#开头*/
		color: red;
	}
	```
	
---
#### 伪类和伪元素
关于伪类和伪元素的解释可以看[这里](http://swordair.com/origin-and-difference-between-css-pseudo-classes-and-pseudo-elements/).

简单概括一下就是

比如下面的例子,想要修改列表中某一行的样式,添加class可以实现,所以:nth-child()是伪类.

再下面想要修改无序列表前的点的样式,正常需要在li标签中添加一个元素来设置,所以:before是一个伪元素.

* 伪类

	> CSS 伪类用于向某些选择器添加特殊的效果。

	这里直接截了W3CSchool的图,只是列出了一部分

	![image](http://7xr09w.com1.z0.glb.clouddn.com/CSS3%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-24%20%E4%B8%8B%E5%8D%888.37.28.png)

	通过一段同样的HTML代码:

	```
	<ol>
		<li>我是第1行</li>
		<li>我是第2行</li>
		<li>我是第3行</li>
		<li>我是第4行</li>
		<li>我是第5行</li>
	</ol>
	```
	
	简单介绍几个常用的伪类:

	:nth-child()

	```
	li:nth-child(3){
		color: red;
		/*
		1. 元素必须是li
		2. 从父级里面找同级元素的第x个,但是必须是li元素
		*/
	}
	```
	效果如下,匹配第3个同级元素
	![image](http://7xr09w.com1.z0.glb.clouddn.com/CSS3%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-24%20%E4%B8%8B%E5%8D%888.47.47.png)
	
	还可以用n来匹配

	```
	li:nth-child(2n+1){
		color: yellow;
	} 
	```
	匹配所有奇数行的样式.
	
* 伪元素

	> CSS 伪元素用于向某些选择器设置特殊效果。
	
	![image](http://7xr09w.com1.z0.glb.clouddn.com/CSS3%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-24%20%E4%B8%8B%E5%8D%888.37.47.png)

	:before
	
	```
	li:before{
		content: "·";
		color: red;
		display: inline-block;
		width: 20px;
		height: 20px;
		background-color: red;
	}
	```

	可以自定义列表前的显示样式,效果:

	![image](http://7xr09w.com1.z0.glb.clouddn.com/CSS3%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-24%20%E4%B8%8B%E5%8D%888.02.43.png)

	通过上面的例子其实可以看出来,伪类和伪元素的最大的区别在于伪类可以通过添加class来实现,伪元素通过添加一个元素来实现.	
