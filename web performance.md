## web 性能优化总结
### 1. web存储  local storage  session storage
###  2. http 缓存
* 客户端三个属性 max-age no-cache last-modified
* max-age  在cache-control必须设置max-ag,这个值告诉客户端在发起下一次的请求之前要把数据缓存多长时间 单位是秒  如果在有效期 返回304
* max-age设置为0 客户端每次都要验证 副作用:中间服务器仍然以过期的缓存进行响应,只要它们在响应中设置警告标志即可
* 如果阻止中间服务器使用缓存可以no-cache
* no-cache属性在某种意义和设置max-age=0非常相似,no-cache告诉客户端在使用缓存中的数据之前,要和服务器重新验证,同时告诉中间服务器,
	它们不能提供过期内容设置是警告信息
* 如果阻止客户端缓存资源,应该使用no-store属性
* no-store属性会通知客户端和中间服务器,不要把这次请求/响应周期中的信息保存在它们的缓存中（容易被窃听、篡改）
* last-modified,如果没有cache-control的前提,会根据last-modified来决定把数据缓存多长时间

### 3. 服务器缓存
* Memcached Redis
* 减轻服务器负载和提升响应时间


#### 4. 数据库缓存



#### 浏览器DNS
* chrome://net-internals/#dns


## 高性能网站建设指南
1. Accept-Encoding Content-Encoding 压缩 gzip
2. 条件GET请求 基于last-modified浏览器将if-modified-since发送给服务器 如果304使用缓存
3. Expires 指出浏览器是否可以使用组件的缓存副本 就不需要条件GET请求
4. keep-alive 持久链接

#### 1减少http请求
1. 雪碧图
2. 内联图 data:<mediatype>[base64]<data>
3. 合并css  script

#### 2使用CDN

#### 3添加Expires
1. Expires:是web服务器告诉客户端直到某个时间为止,可以组件副本
2. Expires缺点是客户端和服务器严格同步
3. http1.1提供cache-control是max-age指定组件缓存多久,单位秒
4. 如果同时有max-age和Expires,max-age将重写Expires
5. Apache的mod-expires可以
6. cache-control优先权并且明确指出了相对于请求时间所经过的秒数,时钟同步问题就避免了


#### 4压缩
1. Accept-Encoding  Content-Encoding gzip  主要压缩是文本文件 html css js
2. 大于1KB或2KB才要压缩 mod_gzip_minimum_file_size
3. Apache1.3 mod_gzip 通过这个模块可以配置 压缩文件类型  mime
4. Apache2.x mod_deflate
5. 代理缓存 需要服务器在响应头添加 vary:Accept-Encoding 将使得代理存响应的多个版本
6. vary:*  防止浏览器使用缓存的组件
7. cache-control:private 为所有浏览器禁用代理的缓存  
8. 默认情况ETag是不能反映出内容是否压缩 因为代理会提供错误的内容 解决办法禁用ETag

#### 5样式放于顶部
* css默认同步下载

* 可以使用异步下载css文件

				<link rel="stylesheet" href="css.css" media="none" onload="if(media!='all')media='all'">

* 使用非阻塞 CSS 加载字体

			<link rel="stylesheet" href="font.css" media="none" onload="if(media!='all')media='all'">

1. link是XHTML标签，除了加载CSS外，还可以定义RSS等其他事务；@import属于CSS范畴，只能加载CSS。

2. link引用CSS时，在页面载入时同时加载；@import需要页面网页完全载入以后加载。

3. link是XHTML标签，无兼容问题；@import是在CSS2.1提出的，低版本的浏览器不支持。

4. link支持使用Javascript控制DOM去改变样式；而@import不支持。

* import导致白屏 最好禁用

#### 6脚本放在底部
* script会阻挡页面html加载解析
* http1.1规范建议浏览器从每个主机名并行下载两个组件
* 下载脚本时候并行下载被禁用 原因如下:1 脚本可能使用document.write改写页面所以浏览器会等待 2保证脚本顺序执行
* [1](https://segmentfault.com/q/1010000000640869)
* [2](http://ued.ctrip.com/blog/script-defer-and-async.html)

#### 7避免css表达式
* expression()


#### 8使用外部javascrip和CSS
* html一般不会被缓存 单独js css 文件会缓存


#### 9减少DNS查找
* 浏览器DNS缓存  操作系统DNS缓存
* win ipconfig /displaydns  ipconfig /flushdns
* 通过使用keep-alive和较少域名来减少DNS查找


#### 10精简javascrip
* 压缩 混淆 GZIP


#### 11避免重定向
* 重定向类型
* 300 多种选择 基于conetent-type
* 301 永久重定向  请求的网页已被永久移动到新位置。服务器返回此响应时，会自动将请求者转到新位置。您应使用此代码通知搜索引擎蜘蛛网页或网  
			站已被永久移动到新位置

* 302 临时重定向
* 303 see other  1.1
* 304 Not modified
* 305 use proxy
* 307 Temporary Redirect 1.1

* <meta http-equiv="refresh" content="0" url="" >

* 缺少结尾的斜线 会引起不必要的重定向

#### 12删除重复脚本



#### 13配置ETag或移除ETag
* 当网站被宿主于在多于一台服务器上时,ETag可能会阻碍缓存
* ETag Entity Tag是web服务器和浏览器用于确认缓存组件有效性的一种机制
* 服务器在检测组件和原始服务器组件匹配方式:last-modified ETag
* ETag在http1.1引入 如果实体依据user-agent Accept-Language而改变 实体状态可以反映在ETag中
* 如果浏览器必须验证一个组件,会使用if-none-match将ETag传回原始服务器
* ETag会减低缓存的效率 if-none-match 比 if-modified-since 的优先级更高


#### 14使Ajax可缓存
* 使用ssl,ssl响应是可缓存的


### 提高用户体验  将图片高度模糊的图片,到原始图片加载完毕替换

### 通过块编码 http1.1引入transfer-encoding：chunked 浏览器下载到数据立刻解析 同时配合使用 Trailer:cookie Trailer:ETag

### 
