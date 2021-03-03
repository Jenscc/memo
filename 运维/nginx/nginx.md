 

# Nginx



## 1.安装



## 2.配置文件



### nginx.conf

#### main(全局模块)

```bash
# 指定用户
#user  nobody; 
# 指定进程数，一般为核心数的两倍
worker_processes  1; 
# 日志记录
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
# 进程ID
#pid        logs/nginx.pid;
```



#### events(nginx工作模式)

```bash
events {
	# 指定运行模型
	use epoll;
    worker_connections  1024;
}
```



#### http(http设置)

upstream(负载均衡服务器设置)

server(主机设置)

location(url 匹配)



```bash
http {
	# include：来设定文件的mime类型(MIME (Multipurpose Internet Mail Extensions) 是描述消息内容类型的因特网标准。)，类型在配设置文件目录下的mime.type文件定义，来告诉nginx来识别文件类型
    include       mime.types;
    
    # default_type：设定了默认的类型问二进制流，也就是当文件类型未定义时使用这种方式，例如在没有配置asp的locate环境时，nginx是不予解析的，此时，用浏览器访问asp文件就会出现下载了
    default_type  application/octet-stream;

	# log_format：用于设置日志的格式，和记录哪些参数，这里设置为main，刚好用于access_log来记录这种类型
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

	# 上传下载文件功能
	sendfile        on;
    #tcp_nopush     on;

	# 超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    
    # server模块
    server {
    	# 监听端口
        listen       80;
      	# 虚拟主机名
      	server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		
        location / {
        	# root path
            root   html;
            # 默认文档，如果您指定了多个文件，那么将按照您指定的顺序逐个查找，可以在列表末尾加上一个绝对路径名的文件。
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}
```



## 3.负载均衡

### 轮询

通过统计判断实现负载均衡

```bash
upstream qidao {
	server 127.0.0.1:1001;
	server 127.0.0.1:1002;
	.....
}
```

修改`location`内容

```bash
location / {
            proxy_pass http://qidao;
            #root   html;
            #index  index.html index.htm;
        }

```



### 权重

权重值越大，优先级越高，按照百分比实现负载均衡

```
upstream ~ {
	server 127.0.0.1:1001 weight=10;
	server 127.0.0.1:1002 weight=5;
	.....
}
```



### iphash

根据访问者ip地址的hash来分配服务器

```
upstream ~ {
	ip_hash;
	server 127.0.0.1:1001 weight=1;
	server 127.0.0.1:1002 weight=2;
	.....
}
```



### 最少连接

将请求分配到连接最少的服务器上,理论上绝对平衡

```
upstream ~ {
	least_conn;
	server 127.0.0.1:1001;
	server 127.0.0.1:1002;
	.....
}
```



### fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。需要插件来实现

```
upstream ~ {
	server 127.0.0.1:1001 weight=1;
	server 127.0.0.1:1002 weight=2;
	fair;
	.....
}
```



### 例子

以轮询为例

```
worker_processes  1;

events {
    worker_connections  1024;
}


http {
        upstream qidao{
        # 以本地Tomcat为例
        server localhost:8080;
        server localhost:8081;
    }

    server {
        listen       80;
        server_name  localhost;

        location / {
            proxy_pass http://qidao;
            proxy_redirect default;
        }
}
```



## 4.限流

### 令牌桶算法

![image-20210302091716077](https://gitee.com/jenscc/picgo/raw/master/img/image-20210302091716077.png)

- 令牌以固定速率产生，并缓存到令牌桶中
- 令牌桶放满时，多余的令牌被丢弃
- 请求要消耗等比例的令牌才能被处理
- 令牌不够时，请求被缓存



### 漏桶算法

- 请求存放在漏桶中，以固定速率流出
- 漏桶满后，多余请求将被丢弃



### 中文文档已过时

#### 配置示例（过时）

```
http {
: limit_zone   one  $binary_remote_addr  10m;

: ...

: server {

: ...

: location /download/ {
: limit_conn   one  1;
: }
```



#### limit_zone(过时)

**语法：** *limit_zone zone_name $variable the_size*

**默认值：** *no*

**作用域：** *http*

本指令定义了一个数据区，里面记录会话状态信息。
$variable 定义判断会话的变量；the_size 定义记录区的总容量。

例子：

```
limit_zone   one  $binary_remote_addr  10m;
```

定义一个叫“one”的记录区，总容量为 10M，以变量 $binary_remote_addr 作为会话的判断基准（即一个地址一个会话）。


您可以注意到了，在这里使用的是 $binary_remote_addr 而不是 $remote_addr。

$remote_addr 的长度为 7 至 15 bytes，会话信息的长度为 32 或 64 bytes。 而 $binary_remote_addr 的长度为 4 bytes，会话信息的长度为 32 bytes。

当区的大小为 1M 的时候，大约可以记录 32000 个会话信息（一个会话占用 32 bytes）。



#### limit_conn（过时）

**语法：** *limit_conn zone_name the_size*

**默认值：** *no*

**作用域：** *http, server, location*

指定一个会话最大的并发连接数。 当超过指定的最发并发连接数时，服务器将返回 "Service unavailable" (503)。

例子：

```
limit_zone   one  $binary_remote_addr  10m;

server {
	location /download/ {
	limit_conn   one  1;
}
```

定义一个叫“one”的记录区，总容量为 10M，以变量 $binary_remote_addr 作为会话的判断基准（即一个地址一个会话）。 限制 /download/ 目录下，一个会话只能进行一个连接。 简单点，就是限制 /download/ 目录下，一个IP只能发起一个连接，多过一个，一律503。



### [英文文档]( http://nginx.org/en/docs/ )

#### limit req module

##### Example Configuration

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

    ...

    server {

        ...

        location /search/ {
            limit_req zone=one burst=5;
        }
```

rate设置产生令牌的速率，burst设置缓冲大小

不过，单独使用 **burst** 参数并不实用。假设 **burst=50** ，rate依然为10r/s，排队中的50个请求虽然每100ms会处理一个，但第50个请求却需要等待 50 * 100ms即 5s，这么长的处理时间自然难以接受。

因此，**burst** 往往结合 **nodelay** 一起使用。

**nodelay** 针对的是 burst 参数，**burst=20 nodelay** 表示这20个请求立马处理，不能延迟，相当于特事特办。不过，即使这20个突发请求立马处理结束，后续来了请求也不会立马处理。**burst=20** 相当于缓存队列中占了20个坑，即使请求被处理了，这20个位置这只能按 100ms一个来释放。

这就达到了速率稳定，但突然流量也能正常处理的效果。



### limit_conn_module

```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
server {
	...
	limit_conn perip 10;
	limit_conn perserver 100;
}
```

**limit_conn perip 10** 作用的key 是 **$binary_remote_addr**，表示限制单个IP同时最多能持有10个连接。

**limit_conn perserver 100** 作用的key是 **$server_name**，表示虚拟主机(server) 同时能处理并发连接的总数。

需要注意的是：只有当 **request header** 被后端server处理后，这个连接才进行计数



#### 设置白名单

限流主要针对外部访问，内网访问相对安全，可以不做限流，通过设置白名单即可。利用 Nginx ngx_http_geo_module 和 ngx_http_map_module两个工具模块即可搞定。

在 **nginx.conf** 的 **http** 部分中配置白名单：

```
    geo $limit {
        default 1;
        10.0.0.0/8 0;
        192.168.0.0/24 0;
        172.20.0.35 0;
    }
    map $limit $limit_key {
        0 "";
        1 $binary_remote_addr;
    }
    limit_req_zone $limit_key zone=myRateLimit:10m rate=10r/s;
```

**geo** 对于白名单(子网或IP都可以) 将返回0，其他IP将返回1。

**map** 将 **$limit** 转换为 **$limit_key**，如果是 **$limit** 是0(白名单)，则返回空字符串；如果是1，则返回客户端实际IP。

**limit_req_zone** 限流的key不再使用 **$binary_remote_addr**，而是 **$limit_key** 来动态获取值。如果是白名单，limit_req_zone 的限流key则为空字符串，**将不会限流**；若不是白名单，将会对客户端真实IP进行限流。



### ab测试

#### 安装

```
yum -y install apr-util
yum -y install yum-utils
cd /opt
mkdir abtmp
cd abtmp
yumdownloader httpd-tools*
rpm2cpio httpd-*.rpm | cpio -idmv
# 在/opt/abtmp目录下会生成一个usr目录，ab就在此目录中
```

简单使用

```
# 进入/opt/abtmp/usr/bin
# -n指定请求数 -c指定连接数
./ab -n100 -c10 http://127.0.0.1/
```



## 5.动静分离

在**server**{}段中加入带正则匹配的**location**来指定匹配项针对服务的动静分离

静态页面交给nginx处理

动态请求交给服务器或apache处理

实现整个网站的动静分离，要求如下：

1.前端Nginx收到静态请求，直接从NFS中返回给客户端

2.前端Nginx收到动态请求，通过FastCGI交给服务器处理

----如果得到静态结果直接从NFS取出结果交给Nginx然后返回给客户端

----如果需要数据处理服务器连接数据库后将结果返回给Nginx

3.前端Nginx收到图片请求以.jpg .png .gif等请求交给后端Images服务器处理

```
# 多个服务组
upstream jing {
	server	localhost:8080;
}
upstream dong {
	server	localhost:8081;
}

location /test.html{
	proxy_pass	http://jing;
}

location ~*\.(jpg|gif)$ {				# location匹配将图片交给Image服务器处理
	proxy_pass http://dong;				# Image服务器需要开启web服务
}
```



## 6.镜像服务器

```bash
# 启用缓存到本地
proxy_store on;
# 表示用户读写权限，如果在error中报路径不允许访问，`chmod -R a+rw` 将下面配置的路径改为相应的权限
proxy_store_access user:rw group:rw all:rw
# 此处为文件的缓存路径，这个路径是和url中的文件路径一致的
proxy_temp_path 缓存目录;
# 在上面的配置后，虽然文件被缓存到了本地硬盘上，但每次请求仍会向远端拉取文件，为了避免这种情况，还必须增加：
if ( !-e $request_filename ) {
	proxy_pass http://.....;
}
# 'http://...'为源服务器地址，默认端口80,如监听其他端口，此处要指出
```

整体配置如下

```bash
location / {  				# 这里的location要换成对应的正则匹配表达式
	expires 3d;				# 设置缓存期限
	proxy_set_header Accept-Encoding '';
	root /home/mpeg/nginx;	# 缓存在服务器上的存储位置
	proxy_store on;			# 开启缓存
	proxy_store_access user:rw group:rw all:rw;	# 用户读写权限
	proxy_temp_path	/home/mpeg/nginx;	# 文件的缓存路径，这个路径必须和服务器缓存的文件路径一致的
	if ( !-e $request_filename ) {
		proxy_pass http://192.168.0.1;	# 此处为源服务器地址
	}
}
```



## 7.热备部署 双主模式

**用Nginx做负载均衡，作为架构的最前端或中间层，随着日益增长的访问量，需要给负载均衡做高可用架构，利用keepalived解决单点风险，一旦Nginx宕机能快速切换到备份服务器**

![image-20210302131730846](https://gitee.com/jenscc/picgo/raw/master/img/image-20210302131730846.png)

### keepalived

配置Keepalived高可用

```
cp /etc/keepalived/keepalived.conf keepalived.conf.bak # 备份初始配置文件
```

```
global_defs {
	vrrp_garp_interval 0
	vrrp_gna_interval 0	
}
vrrp_instance_VI_1 {
	state MASTER			# 备用机修改为 BACKUP
	interface enp0s8		# 修改为对应接口
	virtual_router_id 50
	priority 100			# 权重 备用比主机低就可以了
	advert_int 1
	authentication {		# 认证参数 备用和主机保持一致
		auth_type PASS
		auth_pass 1111
	}
	virtual_ipaddress {  	# 虚拟地址
		192.168.56.120
	}
}
```



### nginx服务热备

```
upstream ... {
	server 127.0.0.1:8080;
	server 127.0.0.1:8081 backup;
}
```



## 8.安全认证(通常不会面向用户)

### htppasswd 生成密码文件

```shell
yum install httpd-tools -y
```

htpasswd指令用来创建和更新用于基本认证的用户认证密码文件。htpasswd指令必须对密码文件有读写权限，否则会返回错误码。

htpasswd参数列表：

| 参数 | 参数说明                                     |
| ---- | -------------------------------------------- |
| -b   | 密码直接写在命令行中，而非使用提示输入的方式 |
| -c   | 创建密码文件，若文件存在，则覆盖文件重新写入 |
| -n   | 不更新密码文件，将用户名密码进行标准输出     |
| -m   | 使用MD5算法对密码进行处理                    |
| -d   | 使用CRYPT算法对密码进行处理                  |
| -s   | 使用SHA算法对密码进行处理                    |
| -p   | 不对密码进行加密处理，使用明文密码           |
| -D   | 从密码文件中删除指定用户记录                 |



htpasswd生成Nginx密码文件

```shell
htpasswd -bc /usr/local/nginx/conf/nginxpasswd Securitit 000000
```

若要在已有Nginx密码文件中追加用户，则无需-c参数

```shell
htpasswd -b /usr/local/nginx/conf/nginxpasswd www 111111
```



### 启用安全认证

```
location / {
	root html;
	auth_basic "Restricted";
	auth_basic_user_file /usr/local/nginx/conf/nginxpasswd;
	....
}
```

