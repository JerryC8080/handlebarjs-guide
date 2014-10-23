Handlerbars 不完全指南 初稿
====
目录
---

###初级部分

1. Introduction
2. Simple Expressions
3. Helpers
4. Block Helper
5. Built-in Helper
6. Comments


###高级部分

1. Precompilation
4. API
5. Little Sprite（HTML Escaping、Handlerbars jQuery）
6. Handlebars in nodejs

---

Introduction
---
**Handlebars**是JavaScript一个语义模板库，通过对view和data的分离来快速构建Web模板。它采用"Logic-less template"（无逻辑模版）的思路，能够在加载时被预编译，而不是到了客户端执行到代码时再去编译， 这样可以保证模板加载和运行的速度。Handlebars兼容Mustache，你可以在Handlebars中导入Mustache模板。


Quick Start
---
###Install
Handlerbars的安装十分简单，只需要登录[官网](http://handlebarsjs.com/)下载最新的版本，然后嵌入到网页中去。handlebars是一个纯JS库，因此你可以像使用其他JS脚本一样用script标签来包含handlebars.js

	<script src="bower_components/handlebars/handlebars.js"></script>
	
###Template
Handlebars expressions 是handlebars模板中最基本的单元，使用方法是加两个花括号`{{value}}`, handlebars模板会自动匹配相应的数值，对象甚至是函数。

建立一个模板,ID（或者class）和type很重要，因为你要用他们来获取模板内容 例如：
	
    <!-- Template -->
    <script id="template" type="text/x-handlebars-template">
        <div class="demo">
            <h1>Name: {{name}}</h1>
            <p>Say :{{content}}</p>
            <p>Here missing : {{content.title}}</p>
        </div>
    </script>
	
###Data
Handlebars会根据上下文来自动对表达式进行匹配，如果匹配项是个变量，则会输出变量的值，如果匹配项是个函数，则函数会被调用。 如果没找到匹配项，则没有输出。

Handlebars 支持JSON格式的数据，准备以下测试数据：

	//  JSON数据
    var context = { name: "JerryC", content: "学挖掘机，到蓝翔"};


###Compile and Execute
我们需要获取模板，然后用`Handlebars.compile`进行编译：

    //  读取模板
    var source = $('#template').html();
    
    //  编译模板
    var template = Handlebars.compile(source);
    
    //  装载数据
    var html = template(context);

###Result

	<div class="demo">
        <h1>Name: JerryC</h1>
        <p>Say :学挖掘机，到蓝翔</p>
        <p>Here missing : </p>
    </div>

###完整脚本
请参考：[handlebars-quick-start.html](https://github.com/JerryC8080/handlebarjs-guide/blob/master/examples/quickStart.html)


Expressions
---
### Simplest Expressions
最简单的表达式：`<h1>{{title}}</h1>`

执行的时候，handlebars首先在当前上下文环境中查找叫‘title’的helper，如果helper不存在，然后再查找叫‘title’的值。

### Path
handlebars同时支持以'.'分隔的路径访问和以‘/’分隔的路径访问，也可以用'../'来访问父级属性。
	
	<!-- 以.访问 -->
	<h1>{{author.name}}</h1>	// value='JerryC'
	
	<!-- 以/访问 -->
	<h1>{{author/name}}</h1>	// value='JerryC'
	
	<!-- 以../访问 -->
	{{#with author}}
		<h1>{{../body}}</h1>	// value='学挖掘机，到蓝翔'
	{{/with}}
	
对应JSON数据

	{
    	title: "My First Blog Post!",
  		author: {
    		id: 88,
    		name: "JerryC"
  		},
  		body: "学挖掘机，到蓝翔"  	
  	};
  	
	
handlebars处理的时候，会先从当前上下文环境中找到article，再找title

### HTML-Escaping

如果插入的是一堆html，那就需要使用三个大括号`{{{content}}}`,其中的content就是html内容。

handlebars除了提供`{{{}}}`形式来填充html之外，也提供了`Handlebars.SafeString`函数来处理。

更多见#[HTML Escaping]()

### Keyword
下面的这些是handlebars表达式的关键词，不能作为标识符来使用：

`!` `"` `#` `%` `&` `'` `(` `)` `*` `+` `,` `.` `/` `;` `<` `=` `>` `@` `[` `\` `]` `^` `{` `|` `}` `~` 

例如：`<h1>{{@title}}</h1>`，这样是不允许的。

###Block
有时候当你需要对某条表达式进行更深入的操作时，Blocks就派上用场了，在Handlebars中，你可以在表达式后面跟随一个#号来表示Blocks，然后通过{{/表达式}}来结束Blocks。 如果当前的表达式是一个数组，则Handlebars会“自动展开数组”，并将Blocks的上下文设为数组中的元素。

	<ul>
	{{#programme}}
    	<li>{{language}}</li>
	{{/programme}}
	</ul>
	
有以下json数据

	{
  		programme: [
    		{language: "JavaScript"},
    		{language: "HTML"},
    		{language: "CSS"}
  		]
	}
	
编译模板代码同上…… 上面的代码会自动匹配programme数据并展开数据，渲染DOM后就是这样的

	<ul>
  		<li>JavaScript</li>
  		<li>HTML</li>
  		<li>CSS</li>
	</ul>
	

Helper
---

###Helper 概念


Helper是一个简单的handlebars标识符，Helper跟函数的概念有点像，因为绑定helper的就是一个回调函数，利用`Handlebars.registerHelper`注册一个helper，然后在`{{helper}}`调用helper进行相关的处理。


Handlebars提供一些诸如`if` `unless` `each` `with` `lookup` `log` 内置的helper以外，还允许用户通过`Handlebars.registerHelper`自定义helper。


请看下面例子：

JSON数据

	{
		jerryc:{
			url : "http://huang-jerryc.com",
			text: "Bluesun --The personal Blog"
		}
	}
	
模板

	{{{link jerryc}}}
	
注册helper

	Handlebars.registerHelper('link', function(object) {
  		var url = Handlebars.escapeExpression(object.url),
      		text = Handlebars.escapeExpression(object.text);

  		return new Handlebars.SafeString(
    	"<a href='" + url + "'>" + objecttext + "</a>"
  		);
	});
	
Result

	<a href=‘http://huang-jerryc.com’>Bluesun --The personal Blog</a>
	
这个例子中，注册了一个叫`link`的helper，并且绑定了一个回调函数，把`jerryc`对象传到回调函数里面，然后回调函数进行处理后返回一串html代码。


###Helper 参数 

Helper后面可以跟零个或多个参数（用空格隔开），每个参数都是一个表达式。

参数的类型可以是string、boolean、number、object或者是key-value的形式，同时也支持路径的方式。
高级一点，helper还支持子表达式的写法。

	<!-- 参数可以为零个 -->
	{{link}}
	
	<!-- 参数可以包含一个或多个 -->
	{{link story}}
	
	<!-- 参数可以是string、boolean、number、object类型，并且可以用path的方式传送 -->
	{{link "See more..." story.url}}
	
	<!-- 参数能以key-value方式接收 -->
	{{link "See more..." href=story.url class="story"}}
	
	<!-- helper支持子表达式的写法 -->
	{{outer-helper (inner-helper 'abc') 'def'}}
	
更多helper的内容，见#[Helper]()



Block Helper
---

Built-in Helper
---

Comments
---

