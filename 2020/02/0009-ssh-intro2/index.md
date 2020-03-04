# SSH 原理与实践


OpenSSH是SSH协议的一个免费开源实现，是用于使用SSH协议进行远程登录的主要连接工具。它对所有流量进行加密，以消除窃听、连接劫持等攻击。此外，OpenSSH还提供了一整套安全的隧道功能、多种身份验证方法以及复杂的配置选项。  

OpenSSH软件主要包含以下几种工具：  
- 服务器端由SSH服务(sshd)、sftp-server和ssh-agent组成  
- 客户端使用ssh-keygen、ssh-add、ssh-kyesign、ssh-keyscan等管理密钥  
- 远程操作使用ssh、scp和sftp完成
  
本文主要介绍SSH的工作流程以及OpenSSH部分工具的使用。

## 认证过程
SSH 协议使用对称加密(symmetric encryption)，非对称加密(asymmetric encryption)和哈希(hashing)来保证信息传输的安全。客户端和服务器端的SSH连接过程主要包括三个阶段：  
1. 在客户端进行服务器验证
2. 生成会话密钥(session key)加密所有通信  
3. 客户认证

### 服务器验证
Ssh连接采用客户端-服务器模型(c/s)，客户端首先向服务器发送连接请求，服务器端运行的SSH服务(sshd)默认监听22端口并处理连接请求，这时客户端需要验证服务器的身份。服务器身份验证主要有两种情况：  

**1. 首次连接**  
如果客户端是第一次连接服务器，则要求客户端通过验证服务器的公钥来手动认证服务器。服务器的公钥一般保存在`/etc/ssh/host_key*`，也可以使用ssh-keyscan命令找到。一旦服务器密钥被接受，那么将会被添加到客户端`~/.ssh/know_hosts`文件中。` known_hosts`文件包含有关客户端所有已验证服务器的信息。  
**2. 再次连接**  
如果客户端不是第一次访问要连接的服务器，则将服务器的身份与`known_hosts`文件中先前记录的信息进行匹配以进行验证。  
<br />
服务器的身份需要用户手动进行验证，这样做的目的主要是为了防止中间人攻击(Man-in-the-middle attack, MITM)。

### 生成会话密钥
验证服务器后，服务器端和客户端使用`Diffie-Hellman`算法生成会话密钥(session key)。 该算法的设计方式是，双方在会话密钥的生成中会做出同等贡献。 生成的会话密钥是在客户端和服务器端共享的对称密钥，即双方使用相同的密钥加密和解密。  
### 客户端认证
最后阶段是使用SSH密钥对来验证客户端的身份。顾名思义，SSH密钥对由两个不同目的的密钥组成： 公钥用于加密数据并可以自由分发，私钥用于解密数据，并且永远不会与任何人共享。  对称加密建立后，将对客户端进行身份验证，验证过程如下：  
1. 客户端首先向服务器发送要验证的密钥对的ID。
2. 服务器检查客户端尝试登录的帐户的authorized_keys文件中的密钥ID。
3. 如果在文件中找到匹配该ID的公钥，那么服务器将生成一个随机数，然后使用客户端公钥对随机数进行加密并将加密的消息发送给客户端。
4. 如果客户端拥有正确的私钥，它将解密该消息以获得服务器生成的随机数。
5. 客户端将获得的随机数与共享的会话密钥结合在一起，并计算该值的MD5哈希值。
6. 然后，客户端将此MD5哈希发送回服务器，作为对加密号码消息的答复。
7. 服务器使用相同的共享会话密钥和发送给客户端的原始号码自行计算MD5值。它将自己的计算结果与客户端发回的计算结果进行比较。如果这两个值匹配，则证明该客户端拥有私钥，并且该客户端已通过身份验证。

不对称密钥允许服务器对客户端进行身份验证，因为客户端只有在拥有正确的关联私钥的情况下才能解密消息。  
此外，SSH支持多种身份验证机制，最常见的是**密码认证**和**公钥认证**。上文只说明了公钥认证过程，当公钥认证未通过时，会再进行密码认证，此处不再赘述。  

## 对称加密
我们在前文中介绍过，SSH 协议使用对称加密(symmetric encryption)，非对称加密(asymmetric encryption)和哈希(hashing)来保证信息传输的安全。  
大部分教程对非对称密钥的作用有详细的介绍，但在对称密钥介绍时往往语焉不详，造成很多人的误解。本节主要是希望大家了解一点，**SSH使用对称密钥来加密整个连接，非对称密钥仅用于身份验证**。  

对称加密是一种加密类型，使用同一个密钥加密发给对方的消息、解密从另一方收到的消息。这意味着拥有密钥的任何人都可以加密和解密发送给拥有密钥的其他人的消息。这种加密模式通常称为“shared secret”加密或“secret key”加密。通常，只有一个密钥用于所有操作，或者只有一对密钥，它们之间的关系很容易发现，或者很容易根据一个密钥推导出另一个密钥。

SSH使用对称密钥来加密整个连接。与一些人的假设相反，我们创建的公有/私有非对称密钥对仅用于身份验证，而不用于加密连接。对称加密甚至可以保护密码验证免遭监听。

客户端和服务端为生成对称密钥做出了同等的贡献，并且最终生成的密钥永远不会为外界所知。这个密钥是通过密钥交换算法创建的，这种算法允许服务器和客户端共享某些公共数据，然后使用各自保留的秘密数据进行处理，最终两者独立地到达相同的目的：生成同一密钥。更详细地说明可参考这篇[文章](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process)。  

通过此过程创建的对称加密密钥是基于会话的(session-bassed)，并且构成了服务器与客户端之间发送的数据的实际加密。一旦建立，必须使用此共享密钥对其余数据进行加密。这个阶段完成之后，服务端和客户端的加密会话就被建立了，然后开始进行客户端的验证。  

SSH可以使用多种对称密码系统，包括AES，Blowfish，3DES，CAST128和Arcfour等。服务器和客户端都可以决定其支持的密码列表，按优先顺序排序。最终两者进行协商，客户端列表中的第一个在服务器上可用的选项将被用作两个方向上的加密算法。


## 文件介绍
SSH密钥验证过程涉及客户端和服务器端的多个文件，为了避免混淆，我们来总结一下各个文件的作用。  

**服务端：**  
- `/etc/ssh/sshd_config`  
ssh服务程序(sshd)的配置文件  
<br />  
- `etc/ssh/ssh_host_*`  
ssh服务程序(sshd)启动时自动生成的服务端公钥和私钥文件，也可以通过`dpkg-reconfigure openssh-server
`命令重新生成。共有八个文件，包括四种加密类型：`rsa`、`dsa`、`ecdsa`和`ed25519`（实际使用时服务器会选择其中一种加密类型）。其中`.pub`结尾的是公钥，将写入到客户端的`~/.ssh/known_hosts`文件中，用于验证服务器身份。  
其中私钥文件严格要求权限为600，若不是则sshd服务可能会拒绝启动。  
<br />  
- `~/.ssh/authorized_keys`  
保存的是基于公钥认证机制时客户端用户的公钥。在进行客户端认证时，服务端将读取对应用户目录下的`authorized_keys`文件。  
<br />  

**客户端：**  
- `/etc/ssh/ssh_config`  
客户端的全局配置文件。  
<br />  
- `~/.ssh/config`  
客户端的用户配置文件，生效优先级高于全局配置文件。一般该文件默认不存在，可自行创建。该文件对权限有严格要求，只对所有者有读/写权限，对其他人完全拒绝写权限。  
<br />  
- `~/.ssh/known_hosts`  
保存服务器验证时服务端`host key`的文件，文件内容来源于服务端的`ssh_host_*_key.pub`文件。  
<br />  
- `/etc/ssh/known_hosts`  
全局`host key`保存文件，作用等同于`~/.ssh/known_hosts`。  
<br />  
- `~/.ssh/id_rsa`  
客户端生成的私钥。由ssh-keygen生成。该文件严格要求权限，文件权限不得大于`711`，一般设置为`600`。  
<br />  
- `~/.ssh/id_rsa.pub`  
私钥`id_rsa`的配对公钥。对权限不敏感。当采用公钥认证机制时，该文件内容需要提前复制到服务端的`~/.ssh/authorized_keys`文件中。  
<br />  
- `~/.ssh/rc`  
保存的是命令列表，这些命令在ssh连接到远程主机成功时将第一时间执行，执行完这些命令之后才开始登陆或执行ssh命令行中的命令。  
<br />  
- `/etc/ssh/rc`  
作用等同于`~/.ssh/rc`  

**注意：**  
配置文件主要包括服务端配置文件`/etc/ssh/sshd_config`和客户端配置文件`/etc/ssh/ssh_config`。这两个文件中有很多同名的配置项，但前者是sshd启动时开关性的设置，后者是请求连接时客户端采取的配置。例如，两配置文件都有GSSAPIAuthentication项，在客户端将其设置为no，表示连接时将直接跳过该身份验证机制，而在服务端设置为no则表示sshd启动时不开启GSSAPI身份验证的机制。即使客户端使用了GSSAPI认证机制，只要服务端没有开启，就绝对不可能认证通过。

## 认证实现
前文介绍了公钥认证的过程以及涉及到的文件，接下来主要介绍公钥认证的具体实现步骤。  
公钥认证过程主要包括两个步骤：1)生成密钥对和 2)分发公钥。  

### 生成密钥对
OpenSSH提供了密钥生成工具ssh-keygen。我们在客户端（服务端也行，无所谓在哪生成）执行`ssh-keygen`指令会出现如下提示：  
```bash
ssh-keygen -t rsa 	#-t参数指定算法，通常使用rsa或dsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):  # 输入密钥对保存路径，与-f参数作用相同
Enter passphrase (empty for no passphrase):               # 输入私钥密码，可留空，与-P参数作用相同
Enter same passphrase again:            
Your identification has been saved in /root/.ssh/id_rsa. 
Your public key has been saved in /root/.ssh/id_rsa.pub. 
```
如不指定保存路径，那么生成的密钥对默认保存在`~/.ssh/`目录下。其中，私钥的权限设置为`600`，如果权限过大会导致公钥认证失败。  

### 分发公钥
密钥生成后，我们要将公钥发送到远程服务器对应用户的家目录下，可以使用`ssh-copy-id`命令实现，语法如下：  
```bash
ssh-copy-id [-i [identity_file]] [user@]host
```
- -i 指定要分发的公钥文件  
- user 指定对应的用户名  
  
举个例子，我们将公钥分发到服务器`114.55.93.224`上的`gavin`用户家目录下:  
  
```bash
ssh-copy-id -i .ssh/id_rsa.pub gavin@114.55.93.224
```
而如果ssh服务端的端口不是22，还需要给`ssh-copy-id`传递端口号，格式为`"-p port_num [user@]hostname"`，如 `"-p 2222 gavin@114.55.93.224"`。  

`ssh-copy-id`命令的作用是在目标主机的指定用户的家目录下，检测是否有`~/.ssh`目录，如果没有，则以700权限创建该目录，然后将本地的公钥追加到目标主机指定用户家目录下的`~/.ssh/authorized_keys`文件中。`authorized_keys`文件可以保存多个公钥信息，每个公钥以换行分开。  
<br />
因此，我们也可以直接将公钥文件传输到服务器上，然后手动将公钥追加到对应用户家目录下的`.ssh/authorized_keys`文件中。  
```bash
cat id_rsa >> authorized_keys
```

公钥分发完成后，我们就可以远程连接服务器了。  

## 最佳实践

### 更改默认端口
Linux默认使用22端口进行远程登录，一些人专门用服务器扫描22端口并使用弱口令等进行暴力破解，通过更改22端口可以过滤掉大部分暴力破解的访问。  
<ruby><rb>SSH服务</rb><rt>ssh daemon</rt></ruby>是OpenSSH软件套件中运行在服务器端的守护进程，它的配置文件是`/etc/ssh/sshd_config`，在配置文件中可以修改守护进程监听的端口。  

在修改之前我们先对配置文件进行备份，然后用文本编辑工具打开：  
```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
vim /etc/ssh/sshd_config
```
打开后可以看到如下内容：  
<pre>
	#	$OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

	# This is the sshd server system-wide configuration file.  See
	# sshd_config(5) for more information.

	# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

	# The strategy used for options in the default sshd_config shipped with
	# OpenSSH is to specify options with their default value where
	# possible, but leave them commented.  Uncommented options override the
	# default value.

	#Port 22
	#AddressFamily any
	#ListenAddress 0.0.0.0
	#ListenAddress ::
</pre>
其中22端口被注释掉了。  
为了防止后续端口修改错误导致无法登录，我们先删除`#`保留`Port 22`端口，然后另起一行添加`Port 2222`，修改后的文件如下：
<pre>
	Port 22
	Port 2222
	#AddressFamily any
	#ListenAddress 0.0.0.0
	#ListenAddress ::
</pre>
修改完成后我们保存退出，重启sshd服务使配置生效：  
```bash
systemctl restart sshd
```
或
```bash
service sshd restart
```

重启完成后，我们可以通过`netstat -ntl`或`ss -ntl`命令查看一下端口。  

配置完成后，记得在防火墙和安全组中放行`2222`端口，然后用新端口重新登录。  
如果登录成功，测试正常后，我们就可以注释或删除掉之前保留的`22`端口了。  

### 禁止root登录
修改`/etc/ssh/sshd_config`文件：  
<pre>
PermitRootLogin yes			# 是否允许root用户登录，默认为yes
</pre>
### 禁止口令登录
修改`/etc/ssh/sshd_config`文件：  
<pre>
PasswordAuthentication yes		# 是否使用密码验证，默认为yes，如果使用密钥对验证可以关闭
</pre>

### 待补充  
  
## 参考资料
1. [Understanding the SSH Encryption and Connection Process](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process)
2. [Understanding SSH workflow](https://medium.com/@Magical_Mudit/understanding-ssh-workflow-66a0e8d4bf65)  
3. [SSH命令和SSH服务详解](https://www.cnblogs.com/f-ck-need-u/p/7129122.html)
<!--more-->
