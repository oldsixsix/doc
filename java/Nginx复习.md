# Nginx
## 正向代理
在客户端（浏览器）中配置代理服务器来访问Google
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518101811630.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 反向代理
客户端对代理是无感知的，客户端不需要配置代理服务器。我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后再返回给客户端。此时暴露的是代理服务器的地址，隐藏了真实服务器的IP地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518102439195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 负载均衡
当并发请求量较大的时候，单个服务器处理不了我们只能增加服务器数量，然后将请求分发到各个服务器上。将原先请求集中到单个服务器的情况改为将请求分发到多个服务器上，也就是说将负载分发到不同的服务器上--负载均衡。
把请求平均分配到服务器上
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518103803363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 动静分离
原来的情况是用单服务部署静态资源 html、css和动态资源jsp servlet。其局限性是动态和静态资源都请求tomcat。
动静分离的做法：
在tomcat中只部署动态资源Jsp,servlet等
创建一个专门的静态资源服务器放静态资源服务器
通过这种方式降低原来单一服务器的处理压力
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518104707179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## Nginx配置文件详解
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530112835381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## nginx的进程模型
`master`进程：主进程，用于接收信号指令，将指令传递给worker分配任务
master会监控worker，当worker发生异常，master会启动一个新的worker
`worker`进程：工作进程，主要用来做事情
worker是为master进行服务的

master接收操作人指令，将指令分发给worker，worker用于连接一个个客户端来处理客户端请求。
### woker抢占进制
当客户端请求进入到nginx之后，woker如何和相应的客户端进行连接呢？
nginx中将客户端请求加上一个互斥锁，那个worker抢到了其他的就进程就没法抢占了。
### nginx事件处理

**查看nginx进程情况**

> ps -ef|grep nginx

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530113057864.png)
**修改工作进程数**
在nginx.conf中修改工作进程数
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530113345233.png)
## Nginx常见命令
```java
./nginx -v? 查看帮助文档，可以看到各种命令
ps -ef|grep nginx 查看nginx进程号
pkill -9 nginx  强制停止nginx
./nginx -s stop 快速停止nginx
./nginx -s quit 优雅停止nginx 没有请求的时候关闭，直到连接关闭的时候
./nginx -s reload 重启nginx服务
./nginx -t 验证nginx配置文件是否正确
./nginx -c 指定nginx配置文件
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530145331174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 搭建静态资源服务器
发布一些静态资源进行访问，设置虚拟主机
采用`include`的方式导入虚拟主机配置 `imooc.conf` 文件
```java
# 虚拟主机配置文件，采用include导入
server  {
	listen		8099;
	server_name		localhost;
	
	location	/ {
	root		foodie ; #foodie这样写与html目录同级
	index    index.html;
	}
}
```
在`nginx.conf`文件中添加
```java
	include imooc.conf;
```
**访问静态资源**
http://localhost:8099 默认会访问foodie文件夹下的index.html
**带上匹配路径/imooc**
例如你要访问 foodie中的资源foodie中必须有imooc文件夹才行，因为静态资源服务器转发的时候带上了imooc
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200530154728935.png)
## 使用gzip压缩提高请求效率
提高请求效率，使用gzip压缩会占用cpu内存
```java
# gzip压缩，压缩之后提高资源传输效率
 gzip  on;  #开启压缩
 # 限制最小压缩  小于1字节文件不会进行压缩
 gzip_min_length	1;
 # 压缩比,压缩比越大压缩内容越多，cpu占用越多
 gzip_comp_level	3;
 # 设置压缩文件类型
 gzip_types	text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/json;
```
## location匹配规则
|符号  |含义  |
| --|--|
| = |精确匹配  |
|^~ |开头表示 uri 以某个常规字符串开头，理解为匹配 url 路径即可。nginx 不对 url 做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）|
| ~|开头表示区分大小写的正则匹配|
|~*| 开头表示不区分大小写的正则匹配|
| /|通用匹配，任何请求都会匹配到
### 实例
```java
不会区分大小写
~* \.(GIF|png|bmp|jpg)
严格区分带小写
~ \.(GIF|png|bmp|jpg)
^~ 不使用正则表达式，而是以某个字符串路径开头
```
## DNS解析域名(域名系统)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531110416779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
把域名解析成对应的IP
浏览器输入域名 → DNS(解析) → 访问解析的ip：192.168.1.xx
实质上网络之前的通信都是以ip地址进行的，所以域名在通信之前都要被解析成IP地址
### 使用SwitchHosts模拟域名访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200531112049890.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
在浏览器中访问`www.imooc.com:8099`相当于`localhost:8099`==`127.0.0.1:8099`
用本地域名访问nginx中的资源
## Nginx解决跨域问题
### CORS跨域资源共享
Cross-Origin Resouorce Sharing
允许浏览器向跨域的服务器发起js请求获取响应
**Jsonp、Springboot Cors、Nginx**
### 在Nginx中解决跨域问题
对8099端口设置跨域
```java
# 虚拟主机配置文件，采用include导入
server  {
	listen		8099;
	server_name		localhost;
# nginx中配置跨域,只要域名即ip地址发生改变即为跨域
 #允许跨域请求的域，*代表所有
 add_header 'Access-Control-Allow-Origin' *;
 #允许带上cookie请求
add_header 'Access-Control-Allow-Credentials' 'true';
 #允许请求的方法，比如 GET/POST/PUT/DELETE
add_header 'Access-Control-Allow-Methods' *;
 #允许请求的header
add_header 'Access-Control-Allow-Headers' *;

	location	/  {
	root		foodie ;  #第一种方式
	# alias         foodie/immoc; # 起别名 用户不知道这个路径
	index    index.html;
	}
}
```
## Nginx进行防盗链

```java
# 对源站点进行防盗链验证 以imooc.com结尾的验证
valid_referers *.imooc.com;
 非法引入会进入下方判断
if ($invalid_referer) {
	#return 404;
 #}

```
## Nginx集群负载均衡
Nginx作为网关进行负载均衡处理

```java

#配置上游服务器,配置了三台tomcat服务器
# 192.168.1.173:8080
# 192.168.1.174:8080
# 192.168.1.175:8080
upstream tomcats {
	server 127.0.0.1:8001;
	server 127.0.0.1:8002;
	server 127.0.0.1:9003;
}

# 配置监听端口
server {
		listen							92;
		#配置域名之后，还有在switchHosts中配置域名跳转
		server_name				www.tomcats.com;
		
		location /  {
			# 这样的话，就可以转到上游服务器了
			proxy_pass  http://tomcats;
		
				}
}
```
当访问`www.tomcats.com:92`的时候就会转到 `http://tomcats`即`upstream tomcats`中添加的三个服务端口中
默认是`轮询算法`
### 加权轮询算法配置
权重分配一般用于服务器性能差距较大的情况，给性能较好的服务器分配较高的权重，给性能差的服务器分配较低的权重

```java
upstream tomcats {
#分配权重，例如有13个请求进来 1个分配到8001,4个分配到8002，8个分配到9003
#权重分配一般用于服务器性能差距较大的情况
	server 127.0.0.1:8001  	weight=1;
	server 127.0.0.1:8002		weight=4;
	server 127.0.0.1:9003		weight=8;
}

```
#### upstream指令参数
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601152803343.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
```java
max_conns 节点的最大连接数即可连接线程数
当请求未处理完则线程占用一个连接节点，处理完请求后释放掉该节点给后面请求使用
slow_start（商业版拥有的功能） 缓慢启动时间
使得服务器慢慢的加入集群，便于服务器启动之后进行一些配置监控
当配置了slow_start，会默认将这个服务器的权重从0慢慢上升到正常值，从而达到慢启动的效果
down 节点下线
backup 备用节点
该服务器是备用机，只有在其他服务器挂掉之后，它在能被用户访问到
max_fails 允许的最大失败数
如果失败次数超过max_fails，则将该服务down掉
fail_timeout 超过最大失败数后的等待时间
超过最大失败数的等待时间，等待时间结束后，再尝试请求该服务器。如果失败数超过max_fails，则继续等待fail_timeout，等待完毕后再尝试请求，如此循环
```
#### keepalived提高吞吐量
keepalived： 设置长连接处理的数量
proxy_http_version：设置长连接http版本为1.1
proxy_set_header：清除connection header 信息

```java
	upstream	colony {
		server	xxx.xx.xxx.xx	weight=1;
		keepalive	32;	
	}
 
	server {
		listen	8088;
		server_name	blogspring.cn;
 
		location / {
			proxy_pass	http://colony;
			proxy_http_version	1.1;
			proxy_set_header	Connection "";
		}
	}
```
**对比**
没设置keepalived吞吐量为`151.8/sec`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601151400591.png)
设置keepalived吞吐量为`204.4/sec`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601151441243.png)
### ip_hash负载均衡
当你服务端的一个特定url路径会被同一个用户连续访问时，如果负载均衡策略还是轮询的话，那该用户的多次访问会被打到各台服务器上，这显然并不高效（会建立多次http链接等问题）。甚至考虑一种极端情况，用户需要分片上传文件到服务器下，然后再由服务器将分片合并，这时如果用户的请求到达了不同的服务器，那么分片将存储于不同的服务器目录中，导致无法将分片合并。所以，此类场景可以考虑采用nginx提供的ip_hash策略。既能满足每个用户请求到同一台服务器，又能满足不同用户之间负载均衡。
本质上`根据用户ip取hash值以后根据hash值再分配到特定的一台服务器上，保证同一个用户每次访问，它的连接只会打在同一台服务器上`
#### hash算法
`iphash`算法只去ip地址的前三段内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601153955150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601154007374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
`hash(ip)# node_counts =index` ,ip为用户的请求地址，node_counts--节点数即服务器数量得到的index来访问特定服务器。
注意：当你移除某一台服务器的时候，不要直接删除，而是标记为down，因为hash算法已经根据服务器现在的数量，index计算了ip地址的访问路径，如果直接删除，ip_hash需要重新计算
### 一致性hash算法
#### hash算法的问题
当服务器节点增加或者减少的时候，hash算法对所有请求又需要重新计算，这会给服务器带了一些额外的开销`我们需要解决的是当服务器数量改变的时候对绝大多数用户请求没有影响`
#### 一致性hash算法解决方式
一致性Hash算法也是使用取模的方法，只是，刚才描述的取模法是对服务器的数量进行取模，而`一致性Hash算法是对2^32取模`，什么意思呢？简单来说，一致性Hash算法将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为0-2^32-1（即哈希值是一个32位无符号整形），整个哈希环如下：
我们对`服务器ip地址，用户请求的ip地址`都进行取模放在hash环上如图所示
整个空间按照顺时针方向组织，也就是说0点左侧的第一个点代表2^32-1^， 0和2^32-1^在零点中方向重合，我们把这个由2^32个点组成的圆环称为Hash环。
**访问规则**
用户沿着顺时针方向访问采用就近原则访问其距离最近的服务器节点
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601160310213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
#### 服务器数量改变情况
**增加服务器的情况**
只要一个用户请求发生了改变
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601161146354.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
#### 减少服务器的情况
只有down掉的服务器所连接的用户受到了影响
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601161247478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### url_hash算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200601161942554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 动静分离
静态归nginx管理，用户是从nginx里面去加载读取静态资源的css/js/html..
动态数据：得到数据可能和上次请求的不同
### 动静分离实现方式CDN
是直接将静态资源全部存放在CDN服务器上。JavaScript,CSS以及img文件都是存放在CDN服务器上，将HTML文件一起存放到CDN上之后，可以将静态资源统一放置在一种服务器上，便于前端进行维护；而且用户在访问静态资源时，可以很好利用CDN的优点——CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。
CDN：内容分发网络类似与边缘服务器，从最近的服务器拉取静态资源
### 动静分离实现方式Nginx

## Nginx反向代理实例1
实现效果
**windows浏览器访问 www.123.com跳转到linux系统的tomcat主页上**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518170715528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
1.  在windows系统的host文件中进行域名和ip的重定向
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518170810550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
通过host配置实现
**www.123.com → 58.97.73.127:80 的重定向**
即www.123.com请求代理服务器
````
添加内容
# 虚拟机服务器ip地址          域名地址
192.168.17.129             www.123.com
````
2. 在nginx服务器上配置反向代理
192.168.17.129:80  → http://127.0.0.1:8080 
![](https://img-blog.csdnimg.cn/20200519083640596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## Nginx反向代理实例2
**实现效果**
使用nginx反向代理服务器，根据访问路径跳转到不同端口的服务中
1. 访问 http://192.168.17.129:9001/edu/  直接跳转到 127.0.0.1:8080
2. 访问 http://192.168.17.129:9001/vod/  直接跳转到 127.0.0.1:8081
**准备工作**
3. 准备两个tomcat服务器，一个8080端口，一个8081端口
4. 创建文件夹和测试页面            
5. 找到nginx配置文件，进行反向代理
url正则表达式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519092230549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
`~`表示url路径中包含即进行反向代理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519084531176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## Nginx负载均衡实例
**实现效果**
1. 浏览器地址栏输入 http://192.168.17.129/edu/a.html，达到负载均衡效果平均到8080和8081端口中
2. 准备两台tomcat服务器在webapps目录中创建edu文件夹，在edu文件夹下创建页面a.html用于测试。
3. 在nginx的配置文件中进行负载均衡配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519092810215.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051909403937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 负载均衡的几种方式
1. 轮询（默认方式）
每个请求按照时间顺序分配到不同的后端服务器上，如果后端服务器down掉可以自动剔除。
2. weigh 
指定轮询几率，weight和访问比例成正比通常用于后端服务器性能不均匀的情况下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519095450530.png)
3. ip_hash
每个请求按访问ip的hash结果进行分配，这样每个请求可以固定一个后端服务器，可以解决session的问题例如：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519101610946.png)
4. fair（第三方）
按照后端服务器的响应时间来分配请求，响应时间短的优先分配。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200519101712739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## Nginx动静分离实例
简单的说就是把动态和静态资源请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求和静态请求分开可以理解成**nginx处理静态页面，tomcat处理动态页面**。
