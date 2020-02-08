# Hugo简明教程

  
Hugo是由GO语言实现的静态网站生成器，自称“The world's fastest framework for building websites”。
  
静态网站的好处是快速、安全和易于部署，最主要的是可以利用版本控制系统来进行管理。  
本文介绍了如何使用Hugo快速搭建个人网站以及如何利用免费的github pages进行发布。  

## Step 1: Install Hugo for Win  
在release[<u>下载</u>](https://github.com/gohugoio/hugo/releases) 对应版本的二进制文件，二进制版本的好处是无需安装额外依赖。下载完成后解压获得hugo.exe文件，然后将其所在路径添加到环境变量PATH中，方便在命令行中使用。  

添加成功后，在cmd中输入`hugo version`检查是否安装成功，如果安装成功会输出  
`Hugo Static Site Generator v0.63.2-934EE21F windows/amd64 BuildDate: 2020-01-27T12:14:15Z`。  
  
我们也可以通过执行`hugo -help`查看命令帮助。  

## Step 2: Create a New Site
创建一个新的hugo站点：  

```bash
hugo new site path/to/site
```

该命令会在site目录下创建一个新的hugo站点，文件目录结构如下：  
site/    
├─&ensp;archetypes  
│&emsp;&emsp;└─&ensp;default.md  
├─&ensp;config.toml    
├─&ensp;content  
├─&ensp;data  
├─&ensp;layouts  
├─&ensp;static  
└─&ensp;themes  

* config.toml  
	站点的全局参数配置文件  
* archetypes  
	存放default.md文件，该文件定义了Hugo的markdown文件`前置数据(Front Matter)`的结构，可以理解为markdown的metadata。我们可以自定义该结构文件，然后在config.toml中指定自定义的结构文件。Front Matter支持三种格式，分别为yaml，toml和json。默认生成的default.md文件为yaml格式，至少包括以下3项：  

```YAML
	---  
	title: "{{ replace .Name "-" " " | title }}"  
	date: {{ .Date }}  
	draft: true  
	---  
```  
&emsp;&emsp;`draft`为`true`表示该文章处于草稿状态，不会被渲染和显示，发布时需要改为`false`。  
  
* content  
	存放网页内容的目录，我们编写的markdown文件都存放在该目录中，是Hugo的默认源目录。  

* data  
	data目录用来存放数据文件，一般是json文件，Hugo提供了相关命令可以从data目录下读取相关的文件数据，然后渲染到HTML页面中，将业务数据与模板分离。

* layouts  
	存放自定义的模板文件，Hugo优先使用layouts目录下的模板，未发现再去themes目录下查找。模板是以`.html`文件指明如何将内容视图渲染为静态页面。  
	模板包括列表页面、主页、分类模板、partals、单页模板等。  

* static  
	存放所有的静态内容，如images, css、js、CNAME等。Hugo在渲染时，会将static目录下的文件直接复制到public目录下，不会做任何渲染。

* themes  
	存放网站主题，可以下载多个主题，themes目录下的每个子目录代表了一个主题。可以通过在config.toml中通过参数theme指定主题，即theme目录下的子目录名字，也可以在执行hugo命令渲染时通过增加flag参数–theme=xx指定。


## Step 3: Add a Theme
Hugo允许我们创建自己的主题或者使用预创建的开源主题。使用预创建的主题可以为我们节约大量的时间，避免关注不必要的技术细节而专注于内容的输出。让我们使用预创建的主题快速开始吧！  
  
首先我们挑选一个喜欢的 [<u>hugo主题</u>](https://themes.gohugo.io/) ，例如 [<u>LoveIt</u>](https://themes.gohugo.io/loveit/)，然后将主题下载到themes目录下。  
```bash
cd path/to/site
git init
git clone https://github.com/xx/xx.git themes/LoveIt
```
执行成功后，会在themesm目录下生成主题目录LoveIt。  
添加主题到配置文件中：
```bash
echo 'theme = "LoveIt"' >> config.toml
```
或直接用文本编辑器打开config.toml修改相应的配置。  
  
一个快速简便的配置方法是用`themes/loveit/exampleSite`下的`config.toml`替换`/site`站点目录下默认生成的`config.tmol`。  

## Step 4: Add Some Content
我们可以手动创建内容文件（content files），然后添加metadata，如title和data等。也可以通过下面的命令自动创建草稿：  
```bash
hugo new posts/my-first-post.md
```  
在`content/posts`目录中会生成一篇名为`my-first-post.md`的文章草稿，并自动添加`archetypes/default.md`中的内容。    

## Step 5: Hosting Hugo Site Locally  
在站点目录下执行：  
```bash
hugo server
```
启动服务器后，可以通过[http://localhost:1313/](http://localhost:1313/)访问站点并调试。Hugo支持所谓的LiveReload，相应配置及内容的修改会即刻生效并在浏览器中加载。  

## Step 6: Build Static Pages
在创建静态页面之前，我们需要对`config.toml`进行配置。因为我们准备将该网页托管到github pages上，需要将baseURL修改为"https://yourgithubusername.github.io/"。  
调试无误后，我们在站点目录下执行：  
```bash
hugo
```  
该命令会在站点目录下新建一个public子目录，然后将渲染后的全部站点文件输出到该目录中。  
我们可以将public目录中的文件直接提交到github上以Githbu Pages方式发布，也可以部署到自己的服务器上。  

## Step 7: Hosting on Github Pages  
我们在github中新建一个repo，命名为`yourgithubusername.github.io`，然后将public中的文件push到该仓库中。  
```bash
cd path/to/site/public
git init
git remote add origin https://github.com/yourgithubusername/yourgithubusername.github.io
git add.
git commit -m "your message"
git push origin master
```  
叮！ 通过<u>http://yourgitubusername.github.io</u> 访问你的网站吧。  
  
## Reference
1. [Hugo Documentation](https://s0gohugo0io.icopy.site/documentation/)

<!--more-->
