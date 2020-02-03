# Hugo简明教程


Hugo是由GO语言实现的静态网站生成器，具有简单、易用、高效、快速部署等特点。  
本文介绍了Win10系统下Hugo的安装，通过hugo theme快速生成静态网页以及如何托管到github pages等内容。  

## Step 1: Install Hugo for Win  
[<u>下载</u>](https://github.com/gohugoio/hugo/releases) 对应版本的二进制文件，解压后获得exe文件，然后将exe所在路径加入环境变量PATH中。  
在cmd中输入`hugo version`检查是否安装成功，通过执行`hugo -help`查看命令帮助。  

## Step 2: Create a New Site
```
hugo new site path/to/site
```

将会在site目录下创建一个新的hugo站点，文件目录结构如下：  
path  
├─&ensp;config.toml  
├─&ensp;archetypes  
│&emsp;&emsp;└─&ensp;default.md    
├─&ensp;content  
├─&ensp;data  
├─&ensp;layouts  
├─&ensp;static  
└─&ensp;themes  

* config.toml  
	站点的全局参数配置文件  
* archetypes  
	存放default.md文件，该文件定义了Hugo的markdown文件前置数据Front Matter的结构。我们可以自定义结构文件，然后在config.toml中指定自定义的结构文件。Front Matter支持三种格式，分别为yaml，toml和json，默认的default.md文件为yaml格式：  

```
	---  
	title: "{{ replace .Name "-" " " | title }}"  
	date: {{ .Date }}  
	draft: true  
	---  
```  	
* content  
	存放网页内容的目录，我们编写的markdown文件都存放在该目录中，是Hugo的默认源目录。  

* data  
	data目录用来存放数据文件，一般是json文件，Hugo提供了相关命令可以从data目录下读取相关的文件数据，然后渲染到HTML页面中，将业务数据与模板分离。

* layouts  
	存放自定义的模板文件，Hugo优先使用layouts目录下的模板，未发现再去themes目录下查找。

* static  
	存放静态文件，比如css、js、img、CNAME等文件目录。Hugo在渲染时，会将static目录下的文件直接复制到public目录下，不会做任何渲染。

* themes  
	存放网站主题，可以下载多个主题，themes目录下的每个子目录代表了一个主题。可以通过在config.toml中通过参数theme指定主题，即theme目录下的子目录名字，也可以在执行hugo命令渲染时通过增加flag参数–theme=xx指定。


## Step 3: Add a Theme
Hugo允许我们创建自己的主题或者使用预创建的开源主题。使用预创建的主题可以为我们节约大量的时间，避免关注不必要的技术细节而专注于内容的输出。让我们使用预创建的主题快速开始吧！  
  
首先我们挑选一个喜欢的 [<u>hugo主题</u>](https://themes.gohugo.io/) ，例如 [<u>loveit</u>](https://themes.gohugo.io/loveit/) 。  
```
	cd path/to/site
	git init
	git clone https://github.com/xx/xx.git themes/loveit
```
将主题下载到themes目录下，执行成功后，会在themesm目录下生成主题目录loveit。
然后将该theme添加到站点的配置中：
```shell
echo 'theme = "loveit"' >> config.toml
```
或直接用文本编辑器打开config.toml修改相应的配置。  
  
一个快速简便的配置方法是用`themes/loveit/exampleSite`下的`config.toml`替换`/site`站点目录下的`config.tmol`。

## Step 4: Add Some Content
我们可以手动创建内容文件（content files），然后添加metadata，如title和data等。也可以通过下面的命令自动创建草稿：  
```shell
hugo new posts/my-first-post.md
```  
在`content/posts`目录中会自动以`archetypes/default.md`为模板，生成一篇名为`my-first-post.md`的文章草稿。  
metadata中的`draft: true`表示该文章处于草稿状态，不会被显示，因此在生成网页前我们需要将其改为`draft: false`。

## Step 5: Hosting Hugo Site Locally  
在站点目录下执行：  
```shell
hugo server
```
启动服务器后，可以通过[http://localhost:1313/](http://localhost:1313/)访问站点并调试。Hugo支持所谓的liveload，相应配置及内容的修改会即刻生效并显示。

## Step 6: Build Site  
调试无误后，我们在站点目录下执行：
```shell
hugo
```  
该命令会在站点目录下生成public子目录，然后将渲染后的全部站点文件输出到该目录中。  
我们可以将public目录中的文件直接提交到github上以Githbu Pages方式发布，也可以部署到自己的服务器上。

## Step 7: Hosting on Github Pages  
我们在github中新建一个名为`yourgithubusername.github.io`的仓库，然后将public中的文件push到该仓库中。  
```shell
cd path/to/site/public
git init
git remote add origin https://github.com/yourgithubusername/yourgithubusername.github.io
git add.
git commit -m "your message"
git push origin master
```  
同步完毕后，即可通过 <u>http://yourgitubusername.github.io</u> 访问你的网页了。  
需要注意的是，`config.toml`中的baseURL需要更改为对应的主页地址。

<!--more-->