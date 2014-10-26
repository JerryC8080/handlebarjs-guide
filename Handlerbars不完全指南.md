Handlerbars 不完全指南 初稿
====
目录
---

###初级部分

1. Introduction
2. Expressions
3. Helpers
4. Built-in Helper


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
  	
	

### HTML-Escaping

如果插入的是一堆html，那就需要使用三个大括号`{{{content}}}`,其中的content就是html内容。

handlebars除了提供`{{{}}}`形式来填充html之外，也提供了`Handlebars.SafeString`函数来处理。

更多见#[HTML Escaping]()

### Keyword
下面的这些是handlebars表达式的关键词，不能作为标识符来使用：

`!` `"` `#` `%` `&` `'` `(` `)` `*` `+` `,` `.` `/` `;` `<` `=` `>` `@` `[` `\` `]` `^` `{` `|` `}` `~` 

例如：`<h1>{{@title}}</h1>`，这样是不允许的。

### Comments
Handlebars的注释写法有两个：

	{{! handlebars comments }}
	{{!-- handlebars comments --}}

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


Handlebars提供一些诸如`if` `unless` `each` `with` `lookup` `log` 内置的helper以外，还允许用户通过`Handlebars.registerHelper()`自定义helper。


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
	
###Block Helper
名符其实，是结合了block语法的helper。形如：

	{{#link jerryc}}
		<p>url:{{url}}</p>
		<p>text:{{text}}</p>
	{{/link}}

写功能的时候，block helper和普通得helper是有一些不一样的地方的：

	Handlebars.registerHelper('link', function(object,option) {
        return option.fn(this);	
    });
    
结果是：

	<p>url:http://huang-jerryc.com</p>
	<p>text:Bluesun --The personal Blog</p>
    
`option.fn()`就像Handlebars.compile()函数一样，提供一个数据，返回一串字符串。

而`this`，是当前的上下文环境，换句话说就是传进来的数据`jerryc`

    
###registerHelper()
`registerHelper()`是`Handlebars`其中的一个函数，能够注册一个或多个helper，作用于所有的模版。

注册一个helper的写法：

	Handlebars.registerHelper('foo', function() {});
	
注册多个helper的写法：

	Handlebars.registerHelper({
  		foo: function() {},
  		bar: function() {}
	});
	
###registerHelper()的回调函数

`registerHelper()`的回调函数支持两个参数：

	Handlebars.registerHelper('foo', function(object,option) {});

`object`就是模版对应的对象，没什么好说的。重点在`option`，它会根据helper的类型而不同。

如果是**普通的helper**，`option`的结构是这样的：

	option:{
		data:Object,	//  存放数据，其实就是回调函数的第一个参数object
		hash:Object,	//	hash列表，如果模板中调用helper的时候，传了key-value的参数，就会存到这里来
		name:'foo'		//	helper的名称
	}
	
对应的模版：`{{foo object}}`

如果是**block helper**，`option`的结构是这样的：

	option:{
		data:Object,		//  存放数据，其实就是回调函数的第一个参数object
		hash:Object,		//	hash列表，如果模板中调用helper的时候，传了key-value的参数，就会存到这里来
		name:'foo'			//	helper的名称
		fn:funtion,			//	fn函数就像Handlebars.compile()函数一样，提供一个数据，返回一串字符串。
		inverse:function	//	目前还不知道什么用途
	}
	
对应的模版：`{{#foo object}}{{/foo}}`



Built-in Helper
---

###if helper
`{{#if}}`就你使用JavaScript一样，你可以指定条件渲染DOM，如果它的参数返回`false`，`undefined`, `null`, `""` 或者 `[]` (一个错误的值), Handlebar将不会渲染DOM，如果存在{{#else}}则执行{{#else}}后面的渲染 例如：

	{{#if list}}
		<ul id="list">
    		{{#each list}}
        		<li>{{this}}</li>
    		{{/each}}
		</ul>
	{{else}}
    	<p>{{error}}</p>
	{{/if}}
对应适用json数据

	var data = {
    	list:['HTML5','CSS3',"WebGL"],
   		"error":"数据取出错误"
	}
	
这里`{{#if}}`判断是否存在list数组，如果存在则遍历list，如果不存在输出错误信息。

###unless helper
`{{#unless}}`这个语法是反向的if语法也就是当判断的值为false时他会渲染DOM 例如：

	<div class="entry">
		{{#unless license}}
  			<h3 class="warning">WARNING: This entry does not have a license!</h3>
  		{{/unless}}
	</div>

如果`license`返回一个错误的值，那么Handlebars就会渲染这个模板，结果会是：

	<div class="entry">
  		<h3 class="warning">WARNING: This entry does not have a license!</h3>
	</div>

###each helper
你可以使用内置的{{#each}} helper遍历列表块内容，用this来引用遍历的元素 例如：

	<ul>
    	{{#each name}}
        	<li>{{this}}</li>
    	{{/each}}
	</ul>
对应适用的json数据

	{
    	name: ["html","css","javascript"]
	};
	
这里的`this`指的是数组里的每一项元素，和上面的Block很像，但原理是不一样的这里的name是数组，而内置的each就是为了遍历数组用的，更复杂的数据也同样适用。

###with helper
`{{#with}}`一般情况下，Handlebars模板会在编译的阶段的时候进行context传递和赋值。使用with的方法，我们可以将context转移到数据的一个section里面（如果你的数据包含section）。 这个方法在操作复杂的template时候非常有用。

	<div class="entry">
  		<h1>{{title}}</h1>
  		{{#with author}}
  			<h2>By {{firstName}} {{lastName}}</h2>
  		{{/with}}
	</div>

对应适用json数据

	{
  		title: "My first post!",
  		author: {
    		firstName: "Charles",
    		lastName: "Jolley"
  		}
	}
	
另外，`{{with}}`也可以嵌套`{{else}}`使用：

	{{#with author}}
  		<p>{{name}}</p>
	{{else}}
 		<p class="empty">No content</p>
	{{/with}}


###lookup helper
lookup中文翻译是查找的意思，效果是在给定的父项中查找一个子项，看lookup helper的源码：

    instance.registerHelper('lookup', function(obj, field) {
      return obj && obj[field];
    });
    
根据javascript的&&(逻辑与)运算规则，如果obj不存在，则返回false，如果obj存在，就判断obj[field]是否存在，如果存在就返回obj[field]的值，否则返回false。
    
上个例子看一下会更清楚，给定这么一个json数据：

	{
  		title: "My first post!",
  		author: {
    		firstName: "Charles",
    		lastName: "Jolley"
  		},
  		skill:['HTML5','CSS3','Javascript']
	}
	
那么：

	{{lookup author 'firstName'}}	//	Charles
	{{lookup skill 'HMLT5'}}		//	null
	{{lookup skill 0}}				//	HTML5
	{{lookup this title}}			//	My first post!
	
lookup配合each有一种巧妙的用法，可以遍历数组，输出数组的值：

	{{#each skill}}
		{{lookup ../skill @index}}、	//	@index是skill的索引
	{{/each}}
	
结果是：`HTML5、CSS3、Javascript、`


###log helper
这个log helper很奇怪，我按照官方的做法：

	{{log 'Look up me!'}}

但是没什么用。

去查看log helper的源码：

    instance.registerHelper('log', function(message, options) {
      var level = options.data && options.data.level != null ? parseInt(options.data.level, 10) : 1;
      instance.log(level, message);
    });
    
这个instance.log()方法何许人也？寻找它的真相(387行)：

  	var logger = {
    	methodMap: { 0: 'debug', 1: 'info', 2: 'warn', 3: 'error' },

    	// State enum
    	DEBUG: 0,
    	INFO: 1,
    	WARN: 2,
    	ERROR: 3,
    	level: 3,

    	// can be overridden in the host environment
    	log: function(level, message) {
    	  if (logger.level <= level) {
    	    var method = logger.methodMap[level];
    	    if (typeof console !== 'undefined' && console[method]) {
    	      console[method].call(console, message);
    	    }
    	  }
    	}
  	};
  	__exports__.logger = logger;
  	var log = logger.log;
  	__exports__.log = log;
  	
`log()`的处理估计是想调用`console`来输出信息，但是注意看`if (logger.level <= level) `，在这里`logger.level=3`，而根据`log()`的源码来看，`level`应该是在`[0,3]`这个区间内才能有效。所以第一个if里面的代码是不是永远都运行不到？我把if判断语句改成了`if (logger.level >= level) `，然后log helper就正常运行了，也许这是Handlebars得一个bug，我向作者提交[issue](https://github.com/wycats/handlebars.js/issues/888)了，看后续会怎么样。

目前我用的Handlebars版本：`v2.0.0`



