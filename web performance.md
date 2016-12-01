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

