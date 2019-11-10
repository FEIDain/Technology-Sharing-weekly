# Nginx

### 一、简介

​	Nginx 是一款**轻量级**的 **Web 服务器**/**反向代理**及**电子邮件代理服务器**。其特点是占有**内存少**，**并发能力强**，**异步**的，**多个连接(万级别)可以对应一个进程**，进行响应。基于**事件驱动模型**。

### 二、Apache Nginx Tomcat

​	这三个都是Web server，但Apache HTTP SERVER 和 Nginx应该叫做HTTP server，Tomcat则是一个Application Server。

​	''一个 HTTP Server 关心的是 HTTP 协议层面的传输和访问控制，所以在 Apache/Nginx 上你可以看到代理、负载均衡等功能。客户端通过 HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）,一个 HTTP Server 始终只是把服务器上的文件如实的通过 HTTP 协议传输给客户端。“

​	”而应用服务器，则是一个应用执行的容器。它首先需要支持开发语言的 Runtime（对于 Tomcat 来说，就是 Java），保证应用能够在应用服务器上正常运行。其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大，所以**应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端**。“

#### Apache HTTP SERVER 和 Nginx

##### Apache HTTP SERVER

- 稳定性

##### Nginx

- 轻量：更少的资源、更多的并发连接、更高的效率
- 配置简洁
- 核心区别：Apache是同步多进程模型，一个连接对应一个进程；Nginx是异步的，多个连接（万级别）可以对应一个进程。

##### 共同点

- 本身不支持生成动态界面，但可以通过其他模块来支持

#### Tomcat

- 应用服务器
- 运行在JVM之上
- 和Nginx配合使用

### 三、源码

Nginx源码结构分析<https://www.kancloud.cn/digest/understandingnginx/202599>

#### 基本结构

​	Nginx 是使用一个 **master 进程**来管理多个 **worker 进程**提供服务。master 负责管理 worker 进程，而 worker 进程则提供真正的客户服务，worker 进程的数量一般跟服务器上 CPU 的核心数相同，worker 之间通过一些进程间通信机制实现负载均衡等功能。Nginx 进程之间的关系可由下图表示：

![img](https://box.kancloud.cn/2016-09-01_57c7edce687dc.jpg)

​	![1573282162320](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1573282162320.png)

根据各模块的功能，可把 Nginx 源码划分为以下几种功能，如下图所示：

![img](https://box.kancloud.cn/2016-09-01_57c7edd0bbe26.jpg)

- 核心模块功能：为其他模块提供一些基本功能：字符串处理、时间管理、文件读写等功能；
- 配置解析：主要包括文件语法检查、配置参数解析、参数初始化等功能；
- 内存管理：内存池管理、共享内存的分配、缓冲区管理等功能；
- 事件驱动：进程创建与管理、信号接收与处理、所有事件驱动模型的实现、高级 IO 等功能；
- 日志管理：错误日志的生成与管理、任务日志的生成与管理等功能；
- HTTP 服务：提供 Web 服务，包括客户度连接管理、客户端请求处理、虚拟主机管理、服务器组管理等功能；
- Mail 服务：与 HTTP 服务类似，但是增加了邮件协议的实现；

#### 为什么Nginx性能这么高？

得益于它的事件处理机制： 异步非阻塞事件处理机制：运用了epoll模型，提供了一个队列，排队解决。

#### Nginx是如何实现高并发的

​	”当用户进入nginx服务的时候，每个worker的listenfd变的可读，并且这些worker会抢一个叫accept_mutex的东西，accept_mutex是互斥的，一个worker得到了，其他的worker就歇菜了。而抢到这个accept_mutex的worker就开始“读取请求–解析请求–处理请求”，数据彻底返回客户端之后（目标网页出现在电脑屏幕上），这个事件就算彻底结束。“

​	”nginx用这个方法是底下的worker进程抢注用户的要求，同时搭配“异步非阻塞”的方式，实现高并发量。“

​	在nginx中的work进程中，为了应对高并发场景，采取了Reactor模型（也就是I/O多路复用，NIO）：

- 每一个worker进程通过I/O多路复用处理多个连接请求；
- 为了减少进程切换（需要系统调用）的性能损耗，一般设置worker进程数量和CPU数量一致。

**I/O 多路复用模型：**在 I/O 多路复用模型中，最重要的系统调用函数就是 select（其他的还有epoll等），该方法的能够同时监控多个文件描述符的可读可写情况（每一个网络连接其实都对应一个文件描述符），当其中的某些文件描述符可读或者可写时，select 方法就会返回可读以及可写的文件描述符个数。

**nginx work进程**：使用 I/O 多路复用模块同时监听多个 FD（文件描述符），当 accept、read、write 和 close 事件产生时，操作系统就会回调 FD 绑定的事件处理器，这时候work进程再去处理相应事件，而不是阻塞在某个请求连接上等待。这样就可以实现一个进程同时处理多个连接。

#### 为什么不使用多线程？

因为线程创建和上下文的切换非常消耗资源，线程占用内存大，上下文切换占用cpu也很高，采用epoll模型避免了这个缺点。

#### Nginx是如何处理一个请求的呢？

首先，nginx在启动时，会解析配置文件，得到需要监听的端口与ip地址，然后在nginx的master进程里面。先初始化好这个监控的socket(创建socket，设置addrreuse等选项，绑定到指定的ip地址端口，再listen)

然后再fork(一个现有进程可以调用fork函数创建一个新进程。由fork创建的新进程被称为子进程 )出多个子进程出来。

然后子进程会竞争accept新的连接。此时，客户端就可以向nginx发起连接了。当客户端与nginx进行三次握手，与nginx建立好一个连接后，此时，某一个子进程会accept成功，得到这个建立好的连接的socket，然后创建nginx对连接的封装，即ngx_connection_t结构体

接着，设置读写事件处理函数并添加读写事件来与客户端进行数据的交换。最后，nginx或客户端来主动关掉连接，到此，一个连接就寿终正寝了。

#### 配置文件

​	Nginx 服务启动时会读入配置文件，后续的行为则按照配置文件中的指令进行。Nginx 的配置文件是纯文本文件。

​	Nginx 配置文件是以 block（块）形式组织，每个 block 都是以一个块名字和一对大括号 “{}” 表示组成，block 分为几个层级，整个配置文件为 main 层级，即最大的层级；在 main 层级下可以有 event、http 、mail 等层级，而 http 中又会有 server block，server block中可以包含 location block。即块之间是可以嵌套的，内层块继承外层块。最基本的配置项语法格式是“配置项名  配置项值1  配置项值2  配置项值3  ... ”；

​	每个层级可以有自己的指令（Directive），例如 worker_processes 是一个main层级指令，它指定 Nginx 服务的 Worker 进程数量。有的指令只能在一个层级中配置，如worker_processes 只能存在于 main 中，而有的指令可以存在于多个层级，在这种情况下，子 block 会继承 父 block 的配置，同时如果子block配置了与父block不同的指令，则会覆盖掉父 block 的配置。指令的格式是“指令名 参数1 参数2 … 参数N;”，注意参数间可用任意数量空格分隔，最后要加分号。

​	下图是 Nginx 配置文件通常结构图示。

![img](https://box.kancloud.cn/2016-09-01_57c7edce82018.jpg)

### 四、Nginx能做什么？

#### 反向代理

Nginx 服务器的反向代理服务是其最常用的重要功能，由反向代理服务也可以衍生出很多与此相关的 Nginx 服务器重要功能，比如后面会介绍的负载均衡。

Nginx 主要能够代理如下几种协议，其中用到的最多的就是做Http代理服务器。

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180905232339438-913760288.png)

​	反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。

​	配置 nginx 的配置文件，进行反向代理：

![img](https://pic1.zhimg.com/v2-0fe9abfb619281cceeea9ee2be455fcd_b.jpg)

#### 正向代理

正向代理，是在用户端的。比如需要访问某些国外网站，我们可能需要购买vpn。

并且**vpn是在我们的用户浏览器端设置的**(并不是在远端的服务器设置)。

浏览器先访问vpn地址，vpn地址转发请求，并最后将请求结果原路返回来。

![img](https://images2018.cnblogs.com/blog/466768/201804/466768-20180403082035578-1377801350.png)

#### HTTP服务器

Nginx本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用Nginx来做服务器。Nginx也可以用于前后端分离。

跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对javascript施加的安全限制。

所谓同源是指，域名，协议，端口都相同。浏览器执行javascript脚本时，会检查这个脚本属于哪个页面，如果不是同源页面，就不会被执行。

Nginx在前后端分离框架设计中，既可以作为前端的HTTP访问器，也可以通过简单配置实现负载均衡，还可以通过反向代理配置解决前后端分离的JavaScript跨域问题。

 解决方案：Nginx服务器中，监听同一个域名和端口，不同路径转发到客户端和服务器，把不同端口和域名的限制通过反向代理，来解决跨域问题。

#### 负载均衡

当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

**配置负载均衡**

在配置的 nginx 文件中 的 upstream tomcats 中的 server 后面添加一个 weight, 即可代表权重。权重越高，分派请求的数量就越多。默认权重是 1。![img](https://pic1.zhimg.com/80/v2-26643c93102d30325a8a9dfbe4a0fc78_hd.jpg)

### 五、 Nginx性能调优

对于Nginx的调优，可以大致从如下指令着手：

```text
1. worker_processes 


2. worker_connections


3. Buffers


4. Timeouts


5. Gzip Compression


6. Static File Caching


7. logging
```

**1. worker_processes**

worker_processes表示工作进程的数量，一般情况设置成CPU核的数量即可，一个cpu配置多于一个worker数，对Nginx而言没有任何益处，另外不要忘了设置worker_cpu_affinity，这个配置用于将worker process与指定cpu核绑定，降低由于多CPU核切换造成的寄存器等现场重建带来的性能损耗。 grep processor /proc/cpuinfo | wc -l这个命令会告诉你当前机器是多少核，输出为2即表示2核。

**2. worker_connections**

worker_connections配置表示每个工作进程的并发连接数，默认设置为1024。

可以更新如下配置文件来修改该值： sudo vim /etc/nginx/nginx.conf

```text
worker_processes 1;


worker_connections 1024;
```

**3. Buffers**

Buffers：另一个很重要的参数为buffer，如果buffer太小，Nginx会不停的写一些临时文件，这样会导致磁盘不停的去读写，现在我们先了解设置buffer的一些相关参数： *client_body_buffer_size*:允许客户端请求的最大单个文件字节数 client_header_buffer_size:用于设置客户端请求的Header头缓冲区大小，大部分情况1KB大小足够 *client_max_body_size*:设置客户端能够上传的文件大小，默认为1m large_client_header_buffers:该指令用于设置客户端请求的Header头缓冲区大小

具体可参考配置如下：

```text
client_body_buffer_size 10K;


client_header_buffer_size 1k;


client_max_body_size 8m;


large_client_header_buffers 2 1k;
```

**4. Timeouts**

client_header_timeout和client_body_timeout设置请求头和请求体(各自)的超时时间，如果没有发送请求头和请求体，Nginx服务器会返回408错误或者request time out。 keepalive_timeout给客户端分配keep-alive链接超时时间。服务器将在这个超时时间过后关闭链接，我们将它设置低些可以让Nginx持续工作的时间更长。 send_timeout 指定客户端的响应超时时间。这个设置不会用于整个转发器，而是在两次客户端读取操作之间。如果在这段时间内，客户端没有读取任何数据，Nginx就会关闭连接。

具体可参考配置如下：

```text
client_body_timeout 12;


client_header_timeout 12;


keepalive_timeout 15;


send_timeout 10;
```

**5. Gzip Compression**

开启Gzip，gzip可以帮助Nginx减少大量的网络传输工作，另外要注意gzip_comp_level的设置，太高的话，Nginx服务会浪费CPU的执行周期。

具体可参考配置如下：

```text
gzip             on;


gzip_comp_level  2;


gzip_min_length  1000;


gzip_proxied     expired no-cache no-store private auth;


gzip_types       text/plain application/x-javascript text/xml text/css application/xml;
```

**6. Static File Caching**

```text
location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {


    expires 365d;


}
```

以上的文件类型可以根据Nginx服务器匹配增加或减少。

**7. logging**

access_log设置Nginx是否将存储访问日志。关闭这个选项可以让读取磁盘IO操作更快。 可以修改配置文件将该功能关闭：

```text
access_log off;
```

然后重启Nginx服务：

```text
sudo service nginx restart
```

### 六、uWSGI

uWSGI 是一个 Web 服务器，它实现了 WSGI 协议、uwsgi、http 等协议。Nginx 中HttpUwsgiModule 的作用是与 uWSGI 服务器进行交换。WSGI 是一种 Web 服务器网关接口。它是一个 Web 服务器（如 nginx，uWSGI 等服务器）与 web 应用（如用 Flask 框架写的程序）通信的一种规范。要注意 WSGI / uwsgi / uWSGI 这三个概念的区分。WSGI 是一种通信协议。uwsgi 是一种线路协议而不是通信协议，在此常用于在 uWSGI 服务器与其他网络服务器的数据通uWSGI 是实现了 uwsgi 和 WSGI 两种协议的 Web 服务器。

#### Nginx 和 uWISG 服务器之间如何配合工作的？

 首先浏览器发起 http 请求到 nginx 服务器，Nginx 根据接收到请求包，进行 url 分析，判断访问的资源类型，如果是静态资源，直接读取静态资源返回给浏览器，如果请求的是动态资源就转交给 uwsgi服务器，uwsgi 服务器根据自身的 uwsgi 和 WSGI 协议，找到对应的后台框架，后台框架下的应用进行逻辑处理后，将返回值发送到 uwsgi 服务器，然后 uwsgi 服务器再返回给 nginx，最后 nginx将返回值返回给浏览器进行渲染显示给用户。

![技术分享图片](http://image.mamicode.com/info/201810/20181004153943263167.png)
