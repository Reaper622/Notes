# Nginx 初探

> Nginx 是一个异步框架的 Web 服务器，也可以作为反向代理，负载均衡器和 HTTP 缓存。



### Nginx 的特点

- 更快 ： 单次请求相应会更快，高并发环境有更好的响应速度。
- 高扩展性： Nginx 基于模块化设计，低耦合度，具有良好的可扩展性。
- 高可靠性：每个 worker进程相对独立，master 进程在一个 worker 进程出错时可以快速拉起新的 worker 子进程提供服务。（反向代理）。
- 低内存消耗：一般情况下，10000个非活跃的 `HTTP Keep-Alive` 连接在 Nginx 中仅消耗 `2.5MB` 的内存。 Nginx 伤的并发连接上限取决于内存大小。
- 热部署：master进程与worker进程分离，使得 Nginx 支持热部署，可在不间断服务的前提下，升级 Nginx 的可执行文件。



### Nginx 的反向代理功能

> 互联网时代，网络应用基本都属于CS结构，即 client 端与 server 端。代理就是在 client 与 server 之间增加了一层用于特定功能的服务器（代理服务器）。

#### 正向代理

> 正向代理，即是一个位于客户端与原始服务器之间的服务器，为了获取原始服务器的内容，客户端会想代理发送一个请求并指定目标，然后代理向原始服务器转交请求并将获得的内容返回给客户端。一个简单的例子，我们有时会使用的歪皮恩（梯子）就是一个正向代理，它会把我们访问墙外的一些网页请求，代理到一个可以访问此网站的代理服务器上，代理服务器去请求网页内容后，再转发给我们。

- **正向代理**是为客户端服务的，客户端可以根据正向代理访问到它本来访问不到的资源。
- **正向代理**对客户端是透明的，但对于服务端不是透明的，服务端无法判断自己是获取到客户端的请求还是代理服务器的请求。



[![ugPz1f.png](https://s2.ax1x.com/2019/10/06/ugPz1f.png)](https://imgchr.com/i/ugPz1f)

​                                                    ——使用 @ConardLi 的一幅图做直观的展示

#### 反向代理

> 反向代理，即以代理服务器来接受网络上的请求，然后把请求转发给内部网络的服务器（可以不止一台），并将服务器上的结果返回给网络上的客户端。

- **反向代理 ** 是为服务端服务的，反向代理帮助服务器接受并转发客户端的请求，从而帮助服务器完成负载均衡等功能。
- **反向代理** 对服务端透明，对我们却不透明，我们无法直接通过请求来访问服务器，必须要先通过代理服务器，这也在一定程度上保护了内容服务器的安全。
- **反向代理** 可以把收到的客户端请求均匀地分配到服务器集群中来实现负载均衡，同时 Nginx 拥有服务器心跳检查功能，如果有一台服务器异常，那么代理进来的请求不会再发送到该服务器上，而是直到该服务器恢复正常才会继续向这个服务器进行请求分发，这也大大保证了服务的稳定性。



> 在安装 Nginx 时，强烈推荐使用源码编译的模式安装，这样便于后期模块的添加，同时由于 Nginx 后续会升级版本，所以可以使用配置和程序分离的模式安装。具体规则详见：[Nginx 目录建议](https://xuexb.github.io/learn-nginx/guide/dir.html)



### Nginx的一些变量

> 涉及较多，较全的变量可以去[Nginx变量]([https://echo.xuexb.com/api/dump/path?a=1&%E4%B8%AD%E6%96%87=%E5%A5%BD%E7%9A%84#123](https://echo.xuexb.com/api/dump/path?a=1&中文=好的#123))查看，这里只说一些常用的。

- 服务端相关
  - $server_port 服务器端口号
  - $server_addr 服务器地址 (ip地址)
  - $server_name 服务器名称 (一般也为ip地址)
  - $status HTTP的状态码
- 客户端相关
  - $http_accept 浏览器支持的 MIME 类型
  - $http_accept 浏览器支持的压缩编码
  - $http_cache_control 浏览器缓存
  - $http_user_agent 用户设备标识
- 链接相关
  - $uri  请求中当前的URI 可不同于`​request_uri`,可通过内部重定向。
  - $request_uri 客户端请求参数的原始 URI
  - $request_method 请求方法
  - $args 请求参数，即uri后面的参数部分

### Nginx 的一些正则匹配规则

 ~ 区分大小写匹配   |  ~*不区分大小写匹配

!~  区分大小写不匹配  |  !~* 不区分大小写不匹配

-d 与 !-d 用来判断是否存在目录

-f  与 !-f 用来判断是否存在文件

-e 与 !-e 用来判断是否存在文件或者目录

-x 和 !-x 用来判断文件是否可执行

### Nginx 的基本配置

一个基本的 Nginx 的配置为 (可以去查看安装后的 `nginx.conf` 文件)

```nginx
events { 
	# 配置影响 Nginx 服务器与用户的网络连接
}

http # 可嵌套多个 server 配置代理，缓存，日志定义等绝大多数功能与第三方模块配置。
{
    server # 配置虚拟主机的相关参数
    { 	# location 配置请求的路由，以及页面的各种情况
        location path 
        {
            ...
        }
        location path
        {
            ...
        }
     }

    server # 一个http中可以有多个 server
    {
        ...
    }

}

```



### Nginx 的一些常用功能

#### 解决跨域

跨域问题是我们经常遇到也是我们有点偏为头疼的一个问题，但当我们使用 Nginx 时，我们可以利用代理转发的特性帮助我们解决跨域问题。

例如，我们的网站为`content.app.com`，而服务端为 `server.app.com`,给我们的接口地址为`/api`此时我们通过启动一个Nginx服务器拦截前端跨域的请求，通过代理来访问`server.app.com`。

```nginx
server {
  #监听80端口
	listen 80; 
	server_name content.app.com;
	location /api {
		proxy_pass server.app.com;
	}
}
```

即我们请求`content.app.com/api`会被代理服务器转发给`server.app.com`,而我们在 `content.app.com`访问的就是代理服务器的 `content.app.com`，属于同源访问，就不存在跨域的现象了。



#### 状态码与错误配置

```nginx
server {
		# 访问私密文件时报 403 错误
		location /secret.js {
			return 403;
		}
  
  	# 指定不同状态码转到特别的文件
  	error_page 404 /404.html;
    error_page 500 501 502 503 /error.html;
  	# 此时的响应码为 400
  	error_page 404 = 400 /400.html; 
}
```

> 注意：error_page 后如果加了 = 号，则响应为制定的响应码，默认为 200 ，不加等号则为原错误的状态码。



#### 配置HTTPS 服务

配置 HTTPS 我们需要在编译阶段让 Nginx 开启 http_ssl_module 模块，我们可以使用 `nginx -V`命令来查看

若结果为 `TLS SNI support enabled` 则证明已经开启。

之后我们需要去下载证书，一般证书下载解压后会有多种格式，使用 Nginx 格式，即 `crt`与`key`即可。

```nginx
# 配置 HTTPS

# 配置个http的站点，用来做重定向，当然如果你不需要把 HTTP->HTTPS 可以把这个配置删了
server {
    listen       80;

    # 配置域名
    server_name www.safe.com safe.com;

    # 添加 STS, 并让所有子域支持, 开启需慎重
    add_header strict-transport-security 'max-age=31536000; includeSubDomains; preload';

    # 配置让这些 HTTP 的访问全部 301 重定向到 HTTPS 的,
    rewrite ^(.*) https://www.safe.com$1 permanent;
    # rewrite 后若为 break，即url重写后直接使用当前资源，不再执行location中其他语句，url地址栏不变。
    # rewrite 后若为 last，即url重写后，马上发起一个新的请求，再次进入server 快，完成location匹配，url地址栏不变
    # rewrite 后若为 redirect，则返回302临时重定向，地址栏url显示重定向后的，爬虫不更新url
    # rewrite 后若为 permanent，则进行301永久重定向，地址栏显示重定向后的url，爬虫更新url。
}

# 配置 HTTPS
server {
    # 配置域名
    server_name www.safe.com safe.com;

    # https默认端口
    listen 443;

    # 添加STS, 并让所有子域支持, 开启需慎重
    add_header strict-transport-security 'max-age=31536000; includeSubDomains; preload';

    # https配置
    ssl on;
    ssl_certificate /ssl/ssl.crt;
    ssl_certificate_key /ssl/ssl.key;

    # 其他按正常配置处理即可...
}
```



#### 适配PC与移动端环境

虽然现在很多跨端应用于自适应功能日趋完善，但还是有很多网站是同时存在PC于H5两个站点，因此根据用户的设备来切换站点也是一项常见的需求。这时我们需要使用到内置变量`http_user_agent`来获取到用户的设备信息，从而根据设备来返回不同的节点。

```nginx
location / {
	# 移动端设备匹配
  if ($http_user_agent ~* '(Android|webOS|iPhone|iPad|BlackBerry)') {
    set $mobile_request '1';
  }
  if ($mobile_request = '1') {
    rewrite ^.+ https://mobile.com 
  }
}
```



#### 图片处理

在前端开发中，我们有时会需要不同尺寸的图片，而一些云服务都提供有带链接上添加参数来获取不同大小图片的服务。我们可以通过使用一个非基本模块（需安装）`ngx_http_image_filter_module`来实现。

```nginx
    # 图片缩放处理
    # 这里约定的图片处理url格式：以 mysite-base.com/img/路径访问
    location ~* /img/(.+)$ {
        alias /Users/cc/Desktop/server/static/image/$1; #图片服务端储存地址
        set $width -; #图片宽度默认值
        set $height -; #图片高度默认值
        if ($arg_width != "") {
            set $width $arg_width;
        }
        if ($arg_height != "") {
            set $height $arg_height;
        }
        image_filter resize $width $height; #设置图片宽高
        image_filter_buffer 10M;   #设置Nginx读取图片的最大buffer。
        image_filter_interlace on; #是否开启图片图像隔行扫描
        error_page 415 = 415.png; #图片处理错误提示图，例如缩放参数不是数字
    }

```



#### 防盗链处理

掘金之前也因为被大肆盗链使用图片而开启了防盗链处理，防盗链处理是我们需要开启来保护服务器的一种方式。

```nginx
server {
  # 匹配所有图片
  location ~* \.(gif|jpg|png|bmp)$ {
    # 验证是否为没有来路，或者有来路与域名或白名单进行匹配，也可以写正则,若匹配到字段为0，否则为1
    valid_referers none blocked server_names www.reaper.com;
    # 如果验证不通过则返回403错误
    if ($invalid_referer) {
      return 403; 
    }
  }
}
```



### 总结

对于Nginx的学习是在配置自己的个人博客网站[Reaper Lee](https://www.reaperlee.cn/)时一边学习一遍配置完成的，希望能够帮助一些准备学习或者刚入门Nginx的同学，但对于逐渐全栈化的前端，学会服务端的配置似乎也已经成为了必修的技能。大家也可以关注我的[GitHub](https://github.com/Reaper622),一起学习，共同进步！

> 参考链接：
>
> - [前端开发者必备的Nginx知识 —— ConardLi](https://juejin.im/post/5c85a64d6fb9a04a0e2e038c)
> - [Nginx入门教程](https://xuexb.github.io/learn-nginx/)