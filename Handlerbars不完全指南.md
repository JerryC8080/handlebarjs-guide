Handlerbars 不完全指南 初稿
====
目录
---

###初级部分

1. 简介
2. Quick Start
2. Simple Expressions 	
2. Path
3. Block
4. Helper
5. 内置Helper
7. 注释


###高级部分

1. Precompilation
2. 自定义Helper
3. Block Helper
4. API
5. Little Sprite（HTML Escaping、Handlerbars jQuery）

---

简介
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
请参考：[Handlebars-quick-start.html](https://github.com/JerryC8080/handlebarjs-guide/blob/master/examples/quickStart.html)
Expressions
---

