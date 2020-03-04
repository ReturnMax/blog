# Nginx 配置介绍

Nginx是一款轻量级的Web服务器，也可作为反向代理服务器、邮件服务器等。  

说到服务器 (Server) 通常会想到两种概念：1) 硬件，就是一台机器，有时也称之为「主机」和 2) 软件，主要用来对外提供一些服务，如邮件服务、数据库服务、网页服务等，它们24小时不间断的运行并监听某些固定的端口，等待客户端的连接并作出回应。  

本文主要介绍如何使用Nginx作为Web服务器快速部署静态网站，对许多配置选项并没有做出详细说明，更加详细的说明请参见文末 [参考资料](#参考资料)。   

## 软件安装

在Debian下可以使用`apt-get`安装nginx：  
```bash
apt-get install nginx
```
默认安装位置在`/etc/nginx`。在Ubuntu 18.04系统中，nginx安装完成后会自动启动。如果没有启动，我们可以执行`nginx`命令手动启动。  
<br />
我们通过`ps -aux`命令可以看到nginx的master进程和worker进程，分别由root用户和www-data用户启动。  
Master进程主要负责读取并验证配置文件nginx.conf，管理worker进程；Worker进程负责处理连接和请求；Worker进程的个数由配置文件决定，一般和CPU核心个数相同（有利于进程切换）。  

安装完成后，在浏览器输入服务器的公网IP，正常情况下可以看到nginx的默认欢迎页（确认已放行80端口）。  
  
接下来修改nginx的配置文件，使这个地址指向我们的网站首页而不是ngixn的默认网页。  

## 配置文件
默认情况下，nginx的配置文件是`/etc/nginx/nginx.conf`。Nginx 配置信息主要分为六大部分： main(全局设置)、events(事件设置)、http(HTTP服务器设置)、 sever(虚拟主机设置)、location(URL匹配设置)、upstream(反向代理设置)。  
  
配置文件里大部分内容是注释掉的，文件内容如下：  

<pre>
user www-data www-data;				//配置用户和用户组，用户组可以为空
worker_processes auto;				//允许生成的进程数，通常设置成和cpu核心数量相等
pid /run/nginx.pid;				//指定nginx进程运行文件存放位置
include /etc/nginx/modules-enabled/*.conf;
//引用modules-enabled目录下.conf配置文件

events {					//events块，配置影响nginx服务器或与用户的网络连接
	worker_connections 768;			//设置最大连接数
	# multi_accept on;			//设置一个进程是否同时接受多个网络连接
}

http {						//http块，可以嵌套多个server
	sendfile on;				//开启高效文件传输模式，去掉将数据复制到缓冲区的步骤
	keepalive_timeout 65;			//长连接超时时间，默认单位是秒
	...
	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
	...
	gzip on;				//开启gzip压缩输出，减少网络传输
	...
	server {				//server块，配置虚拟主机的参数
		listen 80;			//server监听端口
		server_name belldrum.com	//虚拟服务名，可设为主机公网IP地址或域名
		location {			//配置请求的路由和页面的处理
		}
	}
}
</pre>

## 部署网站
假设我们的网站根目录为`/home/gavin/blog`。  
我们在http上下文 (http context) 中定义一个虚拟服务器 (Virtual Server)，并添加以下内容：  
<pre>
http {
	server {
		listen 80;			//监听端口
		server_name belldrum.com;	// 虚拟服务器名
		root /home/gavin/blog;		// 网站资源根目录，nginx需要有该目录的读取权限
		index index.html;		// 显示首页
		location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    			root /home/gavin/blog;
		}				// 配置路由
	}
}
</pre>
其中，`listen`指令中如果省略端口，那么默认会使用标准端口；如果省略IP地址将监听所有地址，上面的例子中已省略地址。  
如果有多个虚拟服务器与请求的IP地址和端口相匹配，那么将对照`server_name`指令测试请求的Host头字段。  

修改完成后保存退出，执行`nginx -s reload`重启nginx服务，便可以通过域名或IP地址访问自己的网站了。  
<br />
我们也可以不用修改`nginx.conf`文件，而是通过在该文件中使用`include`指令管理额外的配置文件。上面的例子中已经通过`include`指令引入了`conf.d`、`sites-enable`、`modules-enabled`三个目录下的配置文件。我们可以直接在这些目录下新建一个虚拟主机的配置文件，或者在任意目录下新建，然后在`nginx.conf`的http上下文中用`include`引用。  
<br />
举个例子：  
我们在conf.d目录下新建一个名为`belldrum.com.conf`的配置文件：  
```bash
cd /etc/nginx/conf.d
vim belldrum.com.conf
```
我们在文件中写入以下内容，然后保存退出：

<pre>
server {
	listen 80;
	server_name belldrum.com;
	root /home/gavin/blog;
	index index.html;
	location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    		root /home/gavin/blog;
	}
</pre>  

最后检查`nginx.conf`的http块内是否已经正确引用即可。  
<pre>
http {
	···
	include /etc/nginx/conf.d/*.conf;
	···
}
</pre>

## 配置 SSL
没有配置ssl加密的网站会被大多数浏览器标记为不安全，而且很多授信的证书授权中心也都提供免费的SSL安全证书，因此给自己的网站添加一把小绿锁势在必行。如果你使用的是阿里云，那么在控制面板就可以申请到免费的SSL证书（这不是广告，阿里云，打钱！！）。  
<br />
如何申请证书不是本节重点，假设你已经拥有了一份有效的SSL安全证书，私钥文件和证书文件分别为`belldrum.com.key`和`belldrum.com.pem`。  
<br />
我们在`/etc/nginx`目录下新建一个`cert`文件夹用于存放私钥和证书。网站证书里面包含公钥，它是公开的，会发送给每一个连接服务器的客户；私钥需要保存在有权限限制的文件中，并保证nginx主进程拥有可读权限。准备完成后，我们再次修改`belldrum.com.conf`文件，修改后的内容如下：  

<pre>
server {
	listen 443 ssl;						//监听443端口，并添加ssl参数
	server_name belldrum.com; 
	root /home/gavin/blog;
	index index.html;

	ssl_certificate /etc/nginx/cert/belldrum.com.pem;	//证书路径
	ssl_certificate_key /etc/nginx/cert/belldrum.com.key;	//私钥路径
	
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;

	location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
		root /home/gavin/blog;
	}
}
</pre>

其中`listen`、`ssl_certificate`和`ssl_certificate_key`是必须明确说明的。`ssl_protocols`和`ssl_ciphers`指令用于要求客户端建立连接时，只使用加强版本的SSL/TLS和加密算法，在没有显式配置时它们的默认值如下：  
<pre>
ssl_protocols TLSv1 TLSv1.1 TLSv1.2
ssl_ciphers HIGH：！aNULL：！MD5  
</pre>
我们简单介绍一下什么是SSL/TLS。SSL (Secure Sockets Layer，安全套接字层) 是一种标准安全协议，用于在在线通信中建立Web服务器和浏览器之间的加密链接；TLS (Transport Layer Security) 是SSL协议的升级版，TLS 1.0通常被认为是SSL 3.1，TLS 1.1即为SSL 3.2，TLS 1.2即为SSL 3.3，两者合称为SSL/TLS。我们只要知道它是一种用于加密的安全协议就好了。HTTP协议用来传输数据，SSL/TLS用于加密传输内容，两者结合就是现在所用的HTTPS。  

`HIGH：！aNULL：！MD5`指明了服务器可用的SSL/TLS加密套件 (CipherSuite)，如`!MD5`意为该算法不可用。我们可以使用`openssl ciphers -V 'HIGH:!aNULL:!MD5'`命令来查看一下具体支持的加密套件，根据OpenSSL版本不同可能略有区别。举个例子：  
<pre>
0xC0,0x23 - ECDHE-ECDSA-AES128-SHA256 TLSv1.2 Kx=ECDH Au=ECDSA Enc=AES(128) Mac=SHA256
</pre>
- 名字为 ECDH-ECDSA-AES128-SHA256 的加密套件，用于 TLSv1.2
- 密钥交换算法（Key Exchange）使用ECDH
- 认证算法（Authentication）使用ECDSA
- 加密算法（Encryption）使用AES-128
- 消息认证码算法（Message Authentication Code, MAC）使用SHA256

加密套件是SSL握手中需要协商的一个重要参数。客户端会在`Client Hello`中带上它所支持的加密套件列表，服务端从中选择一个并通过`Server Hello`返回。如果两者支持的加密套件列表没有交集会导致握手失败。服务器在选择算法时会有优先级，是以客户端提供的优先还是服务器端配置的优先。可以通过`ssl_prefer_server_ciphers on;`设置为服务器端优先。  
<br />
最后，保存配置文件并退出，重启nginx服务即可通过`https://belldrum.com`访问网站了。   

## 设置跳转
我们希望当用户使用`http`访问网站时能够自动重定向到`https`，或者更进一步，能够同时实现子域名`www`到`non-www`的重定向。  
<br />
我们可以使用`rewrite`或者`301`重定向实现这种跳转，在这里只介绍`301`重定向的方法。  
我们将`belldrum.com.conf`配置文件修改如下：  
<pre>
server {
	listen 443 ssl;
	server_name belldrum.com; 
	root /home/gavin/blog;
	index index.html;

	ssl_certificate /etc/nginx/cert/belldrum.com.pem;
	ssl_certificate_key /etc/nginx/cert/belldrum.com.key;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_prefer_server_ciphers on

	access_log /var/log/nginx/belldrum.com-access.log;
	error_log /var/log/nginx/belldrum.com-error.log;

	location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
		root /home/gavin/blog;
	}
}

server {	//设置http重定向到https
	listen 80;
	server_name belldrum.com www.belldrum.com;
	return 301 https://belldrum.com$request_uri;
}

server {	//设置www重定向到non-www
        listen 443 ssl;  
        server_name www.belldrum.com;

        ssl_certificate /etc/nginx/cert/belldrum.com.pem;
        ssl_certificate_key /etc/nginx/cert/belldrum.com.key;
        ssl_prefer_server_ciphers on

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;

        return 301 https://belldrum.com$request_uri;
}
</pre>

如果是多个域名的话，`server_name`选项也支持正则表达式。修改完成后保存退出，然后重启nginx服务即可实现跳转。  

## 设置HSTS
### 添加响应头
<ruby><rb>HTTP严格传输安全</rb><rt>HTTP Strict Transport Security</rt></ruby>，即HSTS，是由互联网工程任务组发布的互联网安全策略机制。采用HSTS策略的网站会强制浏览器使用HTTPS而不是HTTP访问当前资源，以减少会话劫持风险、保护网站流量。
<br />  
网站第一次通过HTTPS请求时，服务器响应`Strict-Transport-Security`头，浏览器记录下这些信息然后在规定时间内访问这个网站的请求都会自动把HTTP替换为HTTPS，并忽略其它的跳转设置(如301重定向跳转)。  

当HSTS响应头(Response Header)设置的过期时间到了，后面通过HTTP的访问恢复到正常模式，不会再自动跳转到HTTPS。每次浏览器接收到STS头，它都会更新这个网站的过期时间，防止过期发生。
<br />  
HSTS响应头的语法如下：  
<pre>
Strict-Transport-Security: max-age=<expire-time>[; includeSubDomains][; preload;]	
</pre>
选项说明：  
- `max-age=<expire-time>` 浏览器收到这个请求后，在`<expire-time>`秒内访问该域名下的请求都使用HTTPS  
- `includeSubDomains` 可选项，如果添加这个参数，那么说明此规则也适用于该网站的所有子域名  
- `preload` 可选项，将域名申请添加到 [预加载HSTS列表](#预加载列表)  
<br />  

在nginx中设置HSTS响应头非常简单，只需要在监听443端口的server中加入以下内容：  
<pre>
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
</pre>
- `always`参数确保为所有响应（包括内部生成的错误响应）设置HSTS响应头  

向客户提供HSTS策略后，它将在指定`<expire-time>`秒内缓存信息。在此期间，浏览器拒绝通过未加密的HTTP访问Web服务，同时也拒绝网站证书错误，而不会给用户继续访问的选择。如果指定了`includeSubDomains`参数，那么这些限制也适用于该域名的所有子域名。  

撤消HSTS策略以删除网站或服务的HTTPS版本非常困难。因此在测试阶段，请先使用非常短的`max-age`时间，例如设置为5分钟(`max-age=300`)，并在测试没有问题时逐渐延长为一周、一月或一年(`max-age=31536000`为一年)。
<br />  
修改后的`belldrum.com.conf`配置文件如下：  
<pre>
server {
	listen 443 ssl;		//监听所有ipv4地址
	listen [::]:443 ssl;	//监听所有ipv6地址
	server_name belldrum.com;

	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
 
	root /home/gavin/blog;
	index index.html;

	ssl_certificate /etc/nginx/cert/belldrum.com.pem;
	ssl_certificate_key /etc/nginx/cert/belldrum.com.key;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_prefer_server_ciphers on

	access_log /var/log/nginx/belldrum-access.log;
	error_log /var/log/nginx/belldrum-error.log;

	location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
		root /home/gavin/blog;
	}
}

server {
	listen 80;
	listen [::]:80;	
	server_name belldrum.com www.belldrum.com;
	return 301 https://belldrum.com$request_uri;
}

server {
        listen 443 ssl;  
        listen [::]:443 ssl;
        server_name www.belldrum.com;

        ssl_certificate /etc/nginx/conf.d/cert/3492990_belldrum.com.pem;
        ssl_certificate_key /etc/nginx/conf.d/cert/3492990_belldrum.com.key;
        
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_prefer_server_ciphers on

        return 301 https://belldrum.com$request_uri;
}
</pre>

### 预加载列表
HSTS并不能完美的解决HTTP会话劫持，它也有一些缺点。如果用户在以下情况通过HTTP访问受HSTS保护的网站，仍然容易受到攻击：  
- 从未访问过该网站
- 切换新的浏览器
- 切换新设备或系统
- 删除了浏览器缓存
- 近期没有访问该网站，max-age已过期  

为了解决这个问题，Google维护了一份 [预加载列表](https://hstspreload.org/) (HSTS Preload List)。我们可以手动将自己的域名提交到HSTS预加载列表，然后这个列表会被硬编码到Chrome浏览器中。同时大多数主流浏览器，如 Firefox、Opera、Safari、Safari、IE 11和Edge等也支持该HSTS预加载列表。  
  
提交HSTS预加载列表之前网站需要满足以下几点要求：  
- 提供有效的证书
- 如果正在监听80端口，则需要在同一主机上将HTTP重定向到HTTPS
- 通过HTTPS服务所有的子域名，特别是www子域名
- HSTS头中的参数要求`max-age`至少为31536000(一年)，且必须包含`includeSubDomains`和`preload`参数

需要注意的是，从预加载列表中删除自己的域名非常困难，可能需要几个月的时间才能使用户更新。因此，除非确定可以长期支持整个站点和其子域名的HTTPS服务，否则不要申请加入该列表。  

## HTTPS优化
SSL操作会消耗额外的CPU资源
### 减少运算量
### 加强安全性

## 参考资料



<pre>
add_header Strict-Transport-Security "max-age=2592000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header Content-Security-Policy "default-src 'self' www.google-analytics.com ajax.googleapis.com www.google.com google.com gstatic.com www.gstatic.com connect.facebook.net facebook.com;";
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "origin";
</pre>

<!--more-->
