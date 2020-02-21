# Hugo简明教程2


在上篇文章中，我们介绍了如何使用预创建的主题快速生成自己的静态网页并利用Github免费托管。  

但是使用预创建的主题怎么能满足我们的需求，作为一个geek我们要的是随心所欲为所欲为！  

接下来的教程就让我们从零开始，学习如何撸出一款高度定制化的Hugo主题吧！  

## Content Management
### Basic Concepts  
* Content  
内容就是我们自己以markdown格式撰写的文章，通常我们把单独的文章放到content目录下，将同一类型的文章放到content的子目录下。  
文章可以通过命令行，如`hugo new about.md`生成，也可以手动创建后放入content目录下。  

* Template  
模板保存在两个地方，分别为`site/layouts`目录中和`site/themes/themename/layouts`目录中，其中前者的优先级高于后者，当前者为空时才去寻找后一个目录下的设置。  
模板主要分为以下几种，分别为基础模板（），单页模板（），列表模板（），局部模板（），首页模板以及404页面模板。  

* Static Page  
Static Page = Content + Template  
页面是通过`hugo`命令生成的静态网站中的html页面。Hugo会根据文章的kind属性和固定的规则去找到相应的模板，然后根据模板生成最终的html页面。  
最终我们自己创建的文章和Hugo自动创建的文件的路径会转换成对应的网站的url，文章、页面和url的对应关系为：  
    ```
	└── content
	    ├── _index.md          // [home]            <- https://example.com/**
	    ├── about.md           // [page]            <- https://example.com/about/
	    ├── posts               
	    |   ├── _index.md      // [section]         <- https://example.com/posts/**         
	    |   ├── firstpost.md   // [page]            <- https://example.com/posts/firstpost/
	    |   ├── happy           
	    |   |   ├── _index.md  // [section]         <- https://example.com/posts/happy/**
	    |   |   └── ness.md    // [page]            <- https://example.com/posts/happy/ness/
	    |   └── secondpost.md  // [page]            <- https://example.com/posts/secondpost/
	    └── quote   
	        ├── _index.md      // [section]         <- https://example.com/quote/**           
	        ├── first.md       // [page]            <- https://example.com/quote/first/
	        └── second.md      // [page]            <- https://example.com/quote/second/
	// hugo默认生成的页面, 没有对应的markdown文章
	分类列表页面               // [taxonomyTerm]    <- https://example.com/categories/**
	某个分类下的所有文章的列表  // [taxonomy]        <- https://example.com/categories/one-category **
	标签列表页面               // [taxonomyTerm]    <- https://example.com/tags/**
	某个标签下的所有文章的列表  // [taxonomy]        <- https://example.com/tags/one-tag **
    ```
    **_<small>注意：\_index.md不是必须的, 如果没有找到_index.md，hugo会使用一些默认值。</small>_**  
    页面总体分为两种，分别为`单页(signle page)`和`列表页(list page)`，根据`[]`中标注的页面`属性(kind)`，`single`包括`page`，`list`包括`home`, `section`, `taxonomyTerm`和`taxonomy`。其中，单页为用户撰写的文章，列表为Hugo自动创建的文件。

LoveIt模板：  
\_default/  
home page - baseof.html + summary.html  
posts - baseof.html + section.html  
post - baseof.html + patrial/  
about - baseof.html + single.html  

posts/  
single.html  



在编写模板的时候，一般会先编写一个基础模板。然后将一些常用的公用模板，如导航栏、页首、页脚等，做成单独的子模板，在需要的时候将这些子模板导入基础模板。  
Hugo默认的基础模板页是`_default/baseof.html`。  

  



<!--more-->
