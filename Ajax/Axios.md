# Axios

### 什么是Axios？

Axios是一个基于promise的HTTP库，它可以用在浏览器和node.js中。划重点：Promise！



### Axios的特点

- 是由浏览器端发起的XMLHttpRequests请求
- node端发起http请求
- 它支持Promise API
- 它可以拦截请求和返回
- 它也可以转化请求和返回数据
- 它可以取消请求
- 自动转化json数据
- 客户端可以支持抵御XSRF(跨站请求伪造)



### 如何使用Axios？

使用NPM安装  

```javascript
npm i axios
```



使用cdn引入 

```html
<script src="https://cdn.bootcss.com/axios/0.16.0/axios.min.js"></script>
```



### GET请求

我们可以直接使用axios的get方法

```JavaScript
//发送一个GET请求
axios.get('/get?username=Reaper')
    .then(response => {
    //do something...
})
    .catch(error => {   //捕获错误
    //deat with error...
})
```

当然我们也可以把参数写为一个对象

```javascript
//发送一个GET请求
axios.get('/get',{
    params:{
        username:'Reaper'
    }
})
    .then(response => {
    //do something...
})
    .catch(error => {   //捕获错误
    //deat with error...
})
```



### POST请求

我们可以使用axios的post方法

```javascript
//发送一个POST请求
axios.post('/post',{
    firstName:'Reaper',
    lastName:'Lee'
})
    .then(response => {
    //do something...
})
    .catch(error => {
    //deal with error..
})
```

### 处理并发请求

因为axios使用的是promise这个异步对象，所以我们可以通过axios处理并发

> 并发助手函数：
>
> axios.all(iterable)
>
> axios.spread(callback)

一个简单的例子

```javascript
function getUserName(){
    return axios.get('/username')
}
function getPassWord(){
    return axios.get('/password')
}

axios.all([getUserName(),getPassWord()]) //axios.all会让参数数组中的函数都执行完后在执行下一步
    .then(axios.spread(function(username,password){
    //axios.spread 会将all执行完拿到的响应参数(参数数组)分散到多个参数中
}))
```

### 创建一个Axios实例

可以自定义axios配置来创建一个axios实例

axios.create([config])

```javascript
var instance = axios.create({
    baseURL: 'https://localhost:8080',
    timeout: 1000
})
```



### Axios API

我们也可以对axios进行设置来生成请求，类似于jQuery的$.ajax()

```javascript
//通过配置发送post请求
axios({
    method: 'post', //方法声明
    url: '/post',
    data: {
        firstName: 'Reaper',
        lastName: 'Lee'
    }
})
```

不过我们也可以通过默认配置来发送请求

```javascript
//使用默认方法发送GET请求
axios('/get')
```

#### 请求的配置信息

```javascript
//请求的一些配置
axios({
    method: 'post', //创建请求时使用的方法，默认为get
    
    baseURL: 'http://localhost:8080', //设置基本路径头，它将在请求发送前自动加在url的前面，但当url是一个绝对路径时它便不会加上
    
    url: '/post', //请求服务器的url
    
    //tramsformRequest 允许在向服务器发送前，修改请求的数据
    //只适用于 'PUT' 'POST' 'PATCH' 请求方法
    //数组中的函数必须返回一个字符串或者ArrayBuffer或者 Stream
    transformRequest: [function(data){
        //对data进行转换处理..
        return data;
    }],
    
    //transformResponse 允许响应传递给then/catch之前修改相应数据
    transformResponse: [function(data){
        //对data进行转换处理...
        
        return data;
    }],
    
    headers:{'X-Requested-With':'XMLHttpRequset'},//自定义请求头
    
    //params 是与请求一起发送的URL参数
    //他必须是一个无格式的对象或者URLSearchParams对象
    params:{
        serachParams:123
    },
    
    //paramsSerializer 是一个负责params序列化的函数
    paramsSerializer: params => {
        return Qs.stringify(params, {arrayFormat: 'brackets'})
    },
    
    //data 是作为请求主体被发送的数据
    //它只适用 'PUT' 'POST' 和 'PATCH'方法
    //如果没有设置transformRequset，必须是下列类型之一:
    // -- string,plain object,ArrayBuffer, ArrayBufferView, URLSearchParams
    //-- 浏览器专用: FormData, File, Blob
    //-- Node专用: Stream
    data: {
        fisrtName: 'Reaper',
        lastName: 'Lee'
    },
    
    //timeout 指定请求超时的毫秒数，请求超过timeout的时间将终端请求
    timeout: 1000, //1s无响应将中断请求
    
    //withCredentials 跨域请求时是否需要使用凭证
    withCredentials: false, //默认为false
    
    // `adapter` 允许自定义处理请求，可以方便我们测试
  	// 返回一个 promise 并应用一个有效的响应.
  	adapter: config => {
    	/* ... */
  	},
    
    //auth 使用HTTP基础验证，并且提供凭据
    //他会设置一个Authorization 头信息，覆盖掉headers中设置的Authorization 头信息
    auth: {
        username: 'reaper'.
        password: '123456'
    },
    
    //responseType 表示服务器相应的数据类型
    // 可以为 arraybuffer,blob,document,json,text,stream
    responseType: 'json', //默认为json
    
    // xsrfCookieName 是用作 xsrf token 的值的cookie的名称
  	xsrfCookieName: 'XSRF-TOKEN', // default
  	// xsrfHeaderName 是承载 xsrf token 的值的 HTTP 头的名称
  	xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的
    
  	// onUploadProgress 允许为上传处理进度事件
  	onUploadProgress: function (progressEvent) {
    	// 对原生进度事件的处理
  	},
    
  	// `onDownloadProgress` 允许为下载处理进度事件
  	onDownloadProgress: function (progressEvent) {
    	// 对原生进度事件的处理
  	},
    
    //maxContentLength 定义允许响应内容的最大尺寸
    maxContentLength: 2000,
    
    //validateStatus 定义对于给定HTTP 响应状态码是resolve reject promise。
    //如果返回true(或者null 或undefined)，promise将被resolve。
    //否则，promise将被reject
    validateStatis: status => {
        return status >= 200 && status < 300; //默认
    }
    //maxRedirects 定义在 node.js 中 follow 的最大重定向数目
  	// 如果设置为0，将不会 follow 任何重定向
  	maxRedirects: 5, // 默认的

  	// `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  	// `keepAlive` 默认没有启用
  	httpAgent: new http.Agent({ keepAlive: true }),
  	httpsAgent: new https.Agent({ keepAlive: true }),
        
    // proxy 定义代理服务器的主机名称和端口
  	// auth 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  	// 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定Proxy-Authorization 头。
      proxy: {
        host: '127.0.0.1',
        port: 9000,
        auth: : {
          username: 'mikeymike',
          password: 'rapunz3l'
        }
      }
})
```

#### 响应的结构

一个请求的响应应该包含下列的信息

```json
{
    //data 是服务器提供的响应
    data: {},
    
    //status 是服务器相应的HTTP状态码
    status: 200,
    
    //statusText 来自服务器相应的HTTP状态信息
    statusText: 'OK',
    
    //headers 服务器相应的头
    headerL {},

	//config 为请求提供的配置信息
	config: {}
}
```

#### 配置的默认值

我们可以通过给请求配置默认值来减少我们的一些重复代码的书写

##### 全局axios默认值

```javascript
axios.defaults.baseURL ="http://localhost:8080";
axios.defaults.headers = {'X-Requested-With':'XMLHttpRequset'}
```

##### 自定义实例的默认值

```javascript
//创建实例时设置默认值
var instanceAxios = axios.create({
    baseURL: 'http://localhost:8080'
})

//实例创建后对默认值进行修改
instanceAxios.defaults.baseURL = "http://localhost:8081"
```

#### 配置的优先级

我们可以在多种情况对配置进行设置，所以我们需要有一个优先级的顺序。

顺序为： 请求的 config 参数 > 实例的 defaults 属性  > 库(lib/default.js)的默认值 

一个简单的例子：

```javascript
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();

// 覆写库的超时默认值
// 现在，在超时前，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;

// 为已知需要花费很长时间的请求覆写超时设置
instance.get('/longRequest', {
  timeout: 5000
});
```

### 拦截器

在请求或者响应前被then 或 catch 处理前拦截它们。

```javascript
//添加响应拦截器
axios.interceptors.response.use(function (config) {
    //在发送请求之前做处理
    return config
},function (error) {
    //对请求的错误做处理
    return Promise.reject(error);
});

//添加响应拦截器
axios.interceptors.response.use(function (response){
    //对响应数据做处理
    return response;
},function(error){
    return Promise.reject(error);
});
```

我们也可以移除拦截器

```javascript
var appInterceptor = axios.interceptors.request.use(function(){
    //do something...
})
axios.interceptors.request.eject(appInterceptor);
```

在axios实例上添加拦截器

```javascript
var instance = axios.create();
instance.interceptors.request.use(function() {
    //do something...
})
```

