https、ssl证书的原理？？openssl用法？

正向代理的nignx可以直接安装在客户端？反向代理的nginx需要单独一台服务器  ？其实也可以用一个端口替代吧？

`正向代理和反向代理中，代理做的事情相同吗？`

正向代理中，客户端经过配置，所有的请求[目的ip是服务器]会转到Nginx； Nginx会获取请求的ip 端口 uri，然后自己转发请求到服务端；

反向代理，客户端直接访问代理[目的ip是代理]，代理从服务器获取资源

==//epoll？？？==

http  https仔细学习下！！！！很实用啊；



#### 面试角度的问题梳理

- 简单介绍你对nginx的了解？nginx相比其它web服务器有什么优缺点？
  - nginx是具有高性能  HTTP 和 反向代理 功能的WEB服务器，也是一个 POP3/SMTP/IMAP代理服务器；
  - nginx主要具有：速度快，并发高[多进程和I/O多路复用]，高可靠，轻量级别 等优点 //[配置简单，扩展性好；热部署；成本低 BSD许可]
    - tomcat对态文件和高并发处理能力弱【200-300并发量】；
    - Apache：重量级、不支持高并发
    - Lighttpd：轻量级、高性能，欧美青睐

- ==什么是正向代理，什么是反向代理？==
  - 架设位置不同[客户端  服务端]；
  - 隐藏对象不同【正向隐藏了客户端---即客户端访问的目的地址是服务器[即客户端需要知道服务器的ip，因为代理在客户端]，访问服务端的是代理 ;  反向隐藏了服务端---即客户端访问的目的地址是代理服务器[即服务端需要知道客户端的ip，因为代理在服务端]；
  - 目的不同【正向 解决访问限制问题； 反向 负载均衡和安全防护】
- nginx的核心文件？核心路径？
  - nginx二进制可执行文件、nginx.conf配置文件、error.log、access.log
  - /nginx/sbin 二进制启动文件； /nginx/logs [内含nginx主进程id];  /nginx/html [访问成功、失败的页面]
- nginx启动、重启和停止nginx服务？
  - 信号控制（向master发送信号）：ps -ef | grep nginx获取master的PID； kill -signal PID
  - 命令行控制：/sbin/nginx -h查看支持参数；-c指定nginx配置文件路径；//还有-tc等；
    - 日常使用，直接可以用重启命令  ./nginx  -s quit ;  ./nginx -c  .....;   ./nginx -s reload
- nginx架构（高可靠的原理、平滑升级的原理）
  - 一个master进程和多个worker进程；master接收外界信息，发送给worker，监控worker状态，worker异常退出后，master会重新启用鑫的worker，由worker处理用户请求。
  - 先发送USR2信号，启动新的master和worker；如何发送QUIT给旧的master处理完请求关闭；
- nginx配置文件组成？
  - 默认三大块：全局块、events块、http块
    - http块可以配置多个server块，每个server块可以配置多个location块；
  - 全局块指令
    - user  user1：配置运行work进程的用户及用户组；例如：按照前面的配置只能访问/home/user1目录的权限 //有点迷糊，例子只配置了用户吧？没有用户组？
    - master process ; worker process (建议和cpu核心数保持一致)
    - 其它：daemon、pid、error_log、include

### Nginx简介

#### 背景介绍

![image-20220529224517009](Nginx课程学习-photos/image-20220529224517009.png)

![image-20220529224612503](Nginx课程学习-photos/image-20220529224612503.png)

//协议：标准、规范；

![image-20220529224914471](Nginx课程学习-photos/image-20220529224914471.png)

![image-20220529224956642](Nginx课程学习-photos/image-20220529224956642.png)

//服务端禁掉某一类客户端的访问；客户端请求（指定了服务端?????）-->代理-->服务端； 服务端响应-->代理-->客户端；



![image-20220529225139571](Nginx课程学习-photos/image-20220529225139571.png)

//请求发给代理（请求未指定服务端????），代理分发给不同服务端；

#### 常见服务器对比

![image-20220529225707091](Nginx课程学习-photos/image-20220529225707091.png)

//web服务开发商市场份额占有率；

![image-20220529230830181](Nginx课程学习-photos/image-20220529230830181.png)

![image-20220529231023234](Nginx课程学习-photos/image-20220529231023234.png)

//tomcat并发量大概在200-300; 

![image-20220529231259410](Nginx课程学习-photos/image-20220529231259410.png)

![image-20220529231312913](Nginx课程学习-photos/image-20220529231312913.png)

![image-20220529231329271](Nginx课程学习-photos/image-20220529231329271.png)

#### Nginx的优点

//epoll？？？

![image-20220605223000950](Nginx课程学习-photos/image-20220605223000950.png)

![image-20220605223335130](Nginx课程学习-photos/image-20220605223335130.png)

![image-20220605223459750](Nginx课程学习-photos/image-20220605223459750.png)

![image-20220605223656871](Nginx课程学习-photos/image-20220605223656871.png)

![image-20220605223720660](Nginx课程学习-photos/image-20220605223720660.png)

#### Nginx的功能特性及常用功能

![image-20220605223840153](Nginx课程学习-photos/image-20220605223840153.png)

![image-20220605223856598](Nginx课程学习-photos/image-20220605223856598.png)

![image-20220605223935721](Nginx课程学习-photos/image-20220605223935721.png)

![image-20220605223957581](Nginx课程学习-photos/image-20220605223957581.png)

![image-20220605225049324](Nginx课程学习-photos/image-20220605225049324.png)

![image-20220605225209392](Nginx课程学习-photos/image-20220605225209392.png)

#### Nginx的官方简介

![image-20220605230129034](Nginx课程学习-photos/image-20220605230129034.png)

![image-20220605230204752](Nginx课程学习-photos/image-20220605230204752.png)

![image-20220605230219200](Nginx课程学习-photos/image-20220605230219200.png)

#### Nginx系统环境准备





















#### Nginx代理概述及环境准备

 正向代理在客户端配置（隐藏客户端），反向代理在服务端配置。![image-20220717161933824](Nginx课程学习-photos/image-20220717161933824.png)

![image-20220717162045781](Nginx课程学习-photos/image-20220717162045781.png)

![image-20220717162607170](Nginx课程学习-photos/image-20220717162607170.png)



133

![image-20220717162848504](Nginx课程学习-photos/image-20220717162848504.png)

146

![image-20220717165247364](Nginx课程学习-photos/image-20220717165247364.png)

//uri:端口后面的所有内容

客户端经过配置，所有的请求会转到Nginx

![image-20220717165746326](Nginx课程学习-photos/image-20220717165746326.png)

![image-20220717170305004](Nginx课程学习-photos/image-20220717170305004.png)

#### Nginx反向代理的配置语法

![image-20220717170501197](Nginx课程学习-photos/image-20220717170501197.png)

 

![image-20220717170651584](Nginx课程学习-photos/image-20220717170651584.png)

//133代理上配置，服务器为146

![image-20220717170943552](Nginx课程学习-photos/image-20220717170943552.png)

![image-20220717171047105](Nginx课程学习-photos/image-20220717171047105.png)



![image-20220717182524881](Nginx课程学习-photos/image-20220717182524881.png)

//没有/的话，会自动把location添加到后面；

![image-20220717184612005](Nginx课程学习-photos/image-20220717184612005.png)

 





//不设置请求头的值的话，服务端只能活得代理的ip和端口；





146配置：

![image-20220717190551801](Nginx课程学习-photos/image-20220717190551801.png)

133配置

![image-20220717190445678](Nginx课程学习-photos/image-20220717190445678.png)



![image-20220717190909325](Nginx课程学习-photos/image-20220717190909325.png)

![image-20220814142729336](Nginx课程学习-photos/image-20220814142729336.png)

//133

![image-20220717191425815](Nginx课程学习-photos/image-20220717191425815.png)

//如下，133代理时地址不存在的时候重定向 到146主页，这时ip会跳转为146而不是133；仍然希望隐藏服务端的ip，返回代理服务器的地址；

//146

![image-20220717191830415](Nginx课程学习-photos/image-20220717191830415.png)

 ![image-20220717191849846](Nginx课程学习-photos/image-20220717191849846.png)



//133 对于146服务器上找不到资源的，先跳转到133的默认端口，然后在133的默认端口配置代理146；返回的信息就是133了，而index.html是146服务器上的，服务器的ip被隐藏

![image-20220717192253339](Nginx课程学习-photos/image-20220717192253339.png)

![image-20220717192727094](Nginx课程学习-photos/image-20220717192833169.png)

#### Nginx反向代理实战

服务器1 2 3的内容相同，则需要使用负载均衡；

![image-20220717193903925](Nginx课程学习-photos/image-20220717193903925.png)

![image-20220717193801840](Nginx课程学习-photos/image-20220717193801840.png)

//机器准备，机器有限，用146的不同端口模拟不同的服务器；

![image-20220717194010686](Nginx课程学习-photos/image-20220717194010686.png)

![image-20220717194133875](Nginx课程学习-photos/image-20220717194133875.png)

#### Nginx的安全控制

![image-20220717194650418](Nginx课程学习-photos/image-20220717194650418.png)

![image-20220717194702024](Nginx课程学习-photos/image-20220717194702024.png)

  ![image-20220717195105182](Nginx课程学习-photos/image-20220717195105182.png)

![image-20220717195120803](Nginx课程学习-photos/image-20220717195120803.png)

流量劫持的示例：中间人劫持你的请求，发送给下面的三方服务器；

![image-20220717195330365](Nginx课程学习-photos/image-20220717195330365.png)

#### Nginx添加ssl的支持（支持https请求）

nginx默认不支持https访问 

./nginx -V查看版本信息；

第三步的位置  ~/nignx/core/nginx-版本；//具体看你自己的nginx安装包位置；

第四步： 配置在原来配置的基础上添加http_ssl_module

![image-20220717195737577](Nginx课程学习-photos/image-20220717195737577.png)

![image-20220717203217943](Nginx课程学习-photos/image-20220717203217943.png)

http默认端口80，https默认端口443

![image-20220717203308153](Nginx课程学习-photos/image-20220717203308153.png)

//缓存较为常用；用于提升效率；

![image-2022071720381170](Nginx课程学习-photos/image-20220717203811170.png)

![image-20220717204032389](Nginx课程学习-photos/image-20220717204032389.png)

![](Nginx课程学习-photos/image-20220717203957662.png)

一般会设为on

![image-20220717204126404](Nginx课程学习-photos/image-20220717204126404.png)

#### 生成证书

![image-20220724103221773](Nginx课程学习-photos/image-20220724103221773.png)

![image-20220724103246146](Nginx课程学习-photos/image-20220724103246146.png)

![image-20220724103418342](Nginx课程学习-photos/image-20220724103418342.png)

![image-20220724103342820](Nginx课程学习-photos/image-20220724103342820.png)

证书控制台，证书申请，输入基本信息、域名（提前购买好），   验证 审批 签发后可用；

//第一种生产环境使用较多，学习期间相对麻烦；

![image-20220724104519232](Nginx课程学习-photos/image-20220724104519232.png)

#### 开启SSL实例

![image-20220724105346207](Nginx课程学习-photos/image-20220724105346207.png)

//nginx解压缩包下的conf/nginx.conf有对应的配置；

ctrl  v ,选中，delete;

![image-20220724105900985](Nginx课程学习-photos/image-20220724105900985.png)

![image-20220724105936863](Nginx课程学习-photos/image-20220724105936863.png)

//不安全是因为证书没经过三方验证； 用阿里云的方式生成key  pom文件，则没有这个问题；

![image-20220724110301735](Nginx课程学习-photos/image-20220724110301735.png)



www.baidu.com ，不需要加https前缀，默认的访问方式是https，用rewrite命令：

![image-20220724110540932](Nginx课程学习-photos/image-20220724110540932.png)

#### 反向代理系统调优

![image-20220724111740947](Nginx课程学习-photos/image-20220724111740947.png)



![image-20220724111905757](Nginx课程学习-photos/image-20220724111905757.png)

![image-20220724112040895](Nginx课程学习-photos/image-20220724112040895.png)

![image-20220724112049304](Nginx课程学习-photos/image-20220724112049304.png)

![image-20220724112455907](Nginx课程学习-photos/image-20220724112455907.png)

![image-20220724112544614](Nginx课程学习-photos/image-20220724112544614.png)

![image-20220724112618634](Nginx课程学习-photos/image-20220724112618634.png)

![image-20220724112644951](Nginx课程学习-photos/image-20220724112644951.png)

### Niginx负载均衡

![image-20220724124744346](Nginx课程学习-photos/image-20220724124744346.png)

![image-20220724124514756](Nginx课程学习-photos/image-20220724124514756.png)

//反向代理重点在服务端的内容不一样的情况，负载均衡重点在服务端的内容一样的情况；

#### 负载均衡的原理及处理流程

![image-20220724124914627](Nginx课程学习-photos/image-20220724124914627.png)

![image-20220724125218578](Nginx课程学习-photos/image-20220724125218578.png)

![image-20220724125243598](Nginx课程学习-photos/image-20220724125243598.png)

#### 负载均衡常用的处理方式

![image-20220724125453417](Nginx课程学习-photos/image-20220724125453417.png)

ping命令可以查看域名对应的ip地址；一个域名可以绑定多个ip；

 ![image-20220724125729165](Nginx课程学习-photos/image-20220724125729165.png)

![image-20220724130842505](Nginx课程学习-photos/image-20220724130842505.png)

![image-20220724130907390](Nginx课程学习-photos/image-20220724130907390.png)

DNS有本地缓存； ipconfig/flushdns

![image-20220724131312607](Nginx课程学习-photos/image-20220724131312607.png)

![image-20220724131226756](Nginx课程学习-photos/image-20220724131226756.png)

![image-20220731133852156](Nginx课程学习-photos/image-20220731133852156.png)

![image-20220731134156047](Nginx课程学习-photos/image-20220731134156047.png)

![image-20220731134226770](Nginx课程学习-photos/image-20220731134226770.png)

硬件：贵；不易扩展；但是效率高

![image-20220731134524194](Nginx课程学习-photos/image-20220731134524194.png)

![image-20220731134600178](Nginx课程学习-photos/image-20220731134600178.png)

  

#### Nginx七层负载均衡

![image-20220731134731358](Nginx课程学习-photos/image-20220731134731358.png)

![image-20220731134935290](Nginx课程学习-photos/image-20220731134935290.png)

 ![image-20220731135018531](Nginx课程学习-photos/image-20220731135018531.png)

![image-20220731135220066](Nginx课程学习-photos/image-20220731135220066.png)

![image-20220731144902602](Nginx课程学习-photos/image-20220731144902602.png)

![image-20220731144926101](Nginx课程学习-photos/image-20220731144926101.png)

![image-20220731144955657](Nginx课程学习-photos/image-20220731144955657.png)

![image-20220731145232607](Nginx课程学习-photos/image-20220731145232607.png)

![image-20220731145426792](Nginx课程学习-photos/image-20220731145426792.png)

![image-20220731145524557](Nginx课程学习-photos/image-20220731145524557.png)

除了浏览器也可以用curl命令测试端口；

![image-20220731150100701](Nginx课程学习-photos/image-20220731150100701.png)

![image-20220731150903697](Nginx课程学习-photos/image-20220731150903697.png)

![image-20220731151926250](Nginx课程学习-photos/image-20220731151926250.png)

  ![image-20220731151640905](Nginx课程学习-photos/image-20220731151640905.png)

![image-20220731151748350](Nginx课程学习-photos/image-20220731151748350.png)

![image-20220731152040246](Nginx课程学习-photos/image-20220731152040246.png)

![image-20220731152110962](Nginx课程学习-photos/image-20220731152110962.png)

![image-20220731153043548](Nginx课程学习-photos/image-20220731153043548.png)

![image-20220731153326463](Nginx课程学习-photos/image-20220731153326463.png)

//问题：服务器性能可能不同；

![image-20220731153357324](Nginx课程学习-photos/image-20220731153357324.png)

![image-20220731154024222](Nginx课程学习-photos/image-20220731154024222.png)

//问题：web需要登录的场景，通过session保存登录信息，不同服务器之间的session不共享，则需要登录多次；故需要把单台服务器的请求固定到单台web服务器上；

![image-20220731155133510](Nginx课程学习-photos/image-20220731155133510.png)

![image-20220731155258331](Nginx课程学习-photos/image-20220731155258331.png)

//ip_hash考虑的是登录 信息缓存不命中的问题；但是可能造成负载不均衡，可以把session信息都保存到redis中；

![image-20220731155427470](Nginx课程学习-photos/image-20220731155427470.png)



![image-20220731160125050](Nginx课程学习-photos/image-20220731160125050.png)

![image-20220731160156070](Nginx课程学习-photos/image-20220731160156070.png)

![image-20220731160454393](Nginx课程学习-photos/image-20220731160454393.png)

url_hash针对的是文件缓存命中的问题； 

![image-20220731161344737](Nginx课程学习-photos/image-20220731161344737.png)

![image-20220731161510225](Nginx课程学习-photos/image-20220731161510225.png)

 ![image-20220731162315814](Nginx课程学习-photos/image-20220731162315814.png)

![image-20220731162533585](Nginx课程学习-photos/image-20220731162533585.png)

![image-20220731162855130](Nginx课程学习-photos/image-20220731162855130.png)

vim:  /表示搜索；

![image-20220731163042211](Nginx课程学习-photos/image-20220731163042211.png)

  //注意：有两个nginx文件夹，一个是nginx的源码，make的时候在源码；另一个是安装目录，存放可执行文件；？？？？

![image-20220731165114558](Nginx课程学习-photos/image-20220731165114558.png)

![image-20220731165557074](Nginx课程学习-photos/image-20220731165557074.png)

![image-20220731165937431](Nginx课程学习-photos/image-20220731165937431.png)

#### Nginx四层负载均衡

![image-20220731171123511](Nginx课程学习-photos/image-20220731171123511.png)

![image-20220731172041265](Nginx课程学习-photos/image-20220731172041265.png)

![image-20220731172050405](Nginx课程学习-photos/image-20220731172050405.png)

  ![image-20220731172245794](Nginx课程学习-photos/image-20220731172245794.png)

![image-20220731172316547](Nginx课程学习-photos/image-20220731172316547.png)

![image-20220731172803918](Nginx课程学习-photos/image-20220731172803918.png)

//允许所有服务器访问reids

![image-20220731173210851](Nginx课程学习-photos/image-20220731173210851.png)

启动reids:

![image-20220731173507877](Nginx课程学习-photos/image-20220731173507877.png)

再启动一个redis：

![image-20220731173528758](Nginx课程学习-photos/image-20220731173528758.png)

同样改配置，启动；



关闭redis:

![image-20220731173239647](Nginx课程学习-photos/image-20220731173239647.png)

连接redis，安装redis客户端，在cmd

![image-20220731173327211](Nginx课程学习-photos/image-20220731173327211.png)

四层和七层的负载均衡，在nginx的底层有什么不同吗？？？？？

![image-20220731231029401](Nginx课程学习-photos/image-20220731231029401.png)

https://blog.csdn.net/tongzidane/article/details/125443140

![image-20220731225426649](Nginx课程学习-photos/image-20220731225426649.png)

![image-20220731225651625](Nginx课程学习-photos/image-20220731225651625.png)

![image-20220731225838760](Nginx课程学习-photos/image-20220731225838760.png)

![image-20220731230120851](Nginx课程学习-photos/image-20220731230120851.png)

//四层和七层如果监听同一个端口，生效的应该是四层；

