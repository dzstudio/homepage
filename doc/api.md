## Simple.config

1. 无论在浏览器还是Node环境中，Simple 均为全局对象。
2. 项目启动后，package.json 中的 simple 配置项会被读取并挂载到 Simple.config 属性中。
3. 通过 initialize 项指定启动时需要执行的预处理脚本。
4. 通过 router 项指定路由配置脚本。
5. 浏览器环境中，通过 simple/config.js 加载 Simple.config。
6. Node环境中，通过 server/main.js 加载 Simple.config。

## Application & Component

### 功能

Application对象作为页面处理器，继承自Component，具有完整组件生命周期。可接收上下文 context: {style, name, template} 作为页面渲染参数。通过 render() 方法进行模板渲染，生成html。render 方法会在 setProps 和 setState 方法调用后重新执行一次。在浏览器模式下，render会直接通过模板引擎进行渲染并回调 didMount() 方法。在node模式下，将调用 server/main.js，由node进行渲染并返回response。

### 接口

| 名称             | 输入                                                         | 功能                                                         |
| ---------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| setProps         | props：需要更新的属性，isSync：是否同步刷新，replace：是否直接替换 | 设置props, 只有在application下才可以使用该方法, 并且没有任何事件触发。 |
| replaceProps     | props, isSync                                                | 同setProps，只是会替换之前的属性。                           |
| forceUpdate      | 无                                                           | 强制刷新, 只会调用 didUpdate 回调。                          |
| setState         | 要更新的state                                                | 更新组件内部数据，并触发页面渲染。                           |
| replaceState     | 要替换的state                                                | 替换组件内部数据，并触发页面渲染。                           |
| setHTML          | html：html字符串，isSync：是否同步刷新                       | 以HTML字符串形式设置数据状态, 更新页面。                     |
| render           | target：渲染到的容器                                         | 渲染组件。                                                   |
| action           | name：事件名称，value：值                                    | 触发事件，通过 this.app.emit 来处理。                        |
| getDOMNode       | 无                                                           | 返回DOM下所有带ref属性的Node                                 |
| isMounted        | 无                                                           | 如果组件渲染到了 DOM 中，isMounted() 返回 true。可以使用该方法保证 setState() 和 forceUpdate() 在异步场景下的调用不会出错。 |
| willMount        | 无                                                           | 服务器端和客户端都只调用一次，在初始化渲染执行之前立刻调用。 |
| didMount         | 无                                                           | 在初始化渲染执行之后立刻调用一次，仅客户端有效（服务器端不会调用）。在生命周期中的这个时间点，组件拥有一个 DOM 展现。如果想和其它 JavaScript 框架集成，使用 setTimeout 或者 setInterval 来设置定时器，或者发送 AJAX 请求，可以在该方法中执行这些操作。 |
| willReceiveProps | 无                                                           | 在组件接收到新的 props 的时候调用。在初始化渲染的时候，该方法不会调用。 |
| shouldUpdate     | 无                                                           | 在接收到新的 props 或者 state，将要渲染之前调用。该方法在初始化渲染的时候不会调用，在使用 forceUpdate 方法的时候也不会。 |
| willUpdate       | 无                                                           | 在接收到新的 props 或者 state 之前立刻调用。在初始化渲染的时候该方法不会被调用。 |
| didUpdate        | 无                                                           | 在组件的更新已经同步到 DOM 中之后立刻被调用。该方法不会在初始化渲染的时候调用。 |
| willUnmount      | 无                                                           | 在组件从 DOM 中移除的时候立刻被调用。                        |
| unmount          | 无                                                           | 移除组件。                                                   |



## Router

### 功能

Router模块主要用于Web页面路由，是页面展示的入口。

### 接口

| 名称  | 输入                                                         | 功能                                                         |
| ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| use   | url：路由规则数组，handle：路由处理器，customs：自定义处理器，将会转发至路由处理器 | 注册路由规则。当匹配到路由规则时，对应的路由处理器将会被调用，同时传入经过解析后的http请求参数。node模式下，路由处理器结束后，会自动将响应结果放入context上下文并回传至浏览器。前端模式下则直接更新容器界面。 |
| go    | url：请求URL，context：上下文对象                            | 解析传入的url并匹配已注册的路由规则，开始处理请求。          |
| parse | url：要格式化的URL，parseParams：格式化参数                  | 将URL解析成对象：http://127.0.0.1:5500/core/build/ => {protocol: "http", hostName: "127.0.0.1", port: "5500", path: "/core/build/", fileExt: null, …} |



## Store

### 功能

Store封装了ajax处理器，内置分页机制，使用deferred函数实现异步网络请求。

### 属性

| 属性名        | 类型                          | 功能                                                         |
| ------------- | ----------------------------- | ------------------------------------------------------------ |
| url           | string                        | 指定请求的url。                                              |
| before        | function (deferred, params)   | 发送请求之前的预处理回调, 可以在这里做缓存。                 |
| after         | function (deferred, params)   | 发送之后请求的回调。                                         |
| cache         | bool                          | 是否缓存。                                                   |
| xhr           | object                        | xhr对象。                                                    |
| type          | string [online, inline]       | online时发送http请求，inline时使用mock数据（data属性）。     |
| params        | object                        | 请求参数。                                                   |
| data          | any                           | type为inline时，设置data为模拟数据。                         |
| dataType      | string                        | 接收数据类型，默认为JSON。                                   |
| jsonp         | bool                          | 是否启用jsonp。                                              |
| dataFilter    | function(type, options, json) | ajax层数据过滤，请求完成后，数据处理。                       |
| ajaxOptions   | object                        | ajax相关配置参数。                                           |
| paging        | bool                          | 是否启动分页。                                               |
| context       | object                        | 上下文。                                                     |
| pagingOptions | object                        | 分页配置，默认为：{ map: {limit: 'limit', page: 'page'}, limit: 20, page: 1, min: 1} |
| method        | string                        | 请求类型，默认为'GET'。                                      |
| request       | function                      | 请求处理器，不需要定义，已内置兼容node和浏览器的请求处理器。 |
| abort         | function                      | 取消ajax请求，已实现，不需要自定义。                         |

### 示例

```javascript
module.exports = Simple.Store.extend({ 
    type: 'online',  
    dataType: 'json', 
    jsonp: true, 
    params: {}, 
    url: 'px://api/h5/brand-h5/get-hot-brands.htm', 
    dataFilter: function(type, options, json){ 
      return json; 
    },
});

var ListStore = require('./store/list');
ListStore.create().request({})
    .done(function(data) {})
    .fail(function() {});
```



## Utils

### clone

```javascript
// 重写对象克隆方法, 判断Array或者Object
Simple.Utils.clone(obj)
```

### each

```javascript
// 重写each方法
Simple.Utils.each(properties.mixins, function (v, i) {
    Simple.Utils.each(v, function (v, k) {
        Class.prototype[k] = v;
    });
});
```

### extend

```javascript
// 重写extend方法，方法来自jquery，将一个或多个对象的内容合并到目标对象
Simple.Utils.extend(that.props, props);
```

### join

```javascript
// 数组拼接成字符串
Simple.Utils.joinAll(['a', 'b', 'c'], 'x');
// 输出 axbxc
```

### param

```javascript
// 对象转换成字符串，用于URL拼接
Simple.Utils.param( {a:'abc',b:'xxx',c:'dde'}, true)；
// 输出 a=abc&b=xxx&c=dde，第二个参数表示是否递归处理内部object。
```

### split

```javascript
// 字符串截取函数
Utils.split(param, '&', '=', true)；
//  参数分别为： text 字符串，lineSplitter 分割，fieldSplitter 分割，trim 是否trim
```

### underscore

undersocre是一个javascript函数工具集，提供针对集合和对象等数据的操作工具函数 https://underscorejs.org/ unserscore已集成到项目中，目前的版本为1.8.3，unserscore中提供的函数均可直接通过 Simple.Utils 访问到，例如：Simple.Utils.isArray(obj)

## Tools

### ajax

#### 功能

ajax提供一套异步网络调用方案。Store模块的网络请求最终是通过ajax模块下的http组件发出的，兼容node, jsonp, IE和其他现代浏览器。

#### 示例

```javascript
// 以store/request.js文件为例
var ajax;
ajax = require('../tools/ajax/main')(url, options);
ajax.done(function (data, response, context) {
    that.data = data;
    that.response = response;
    dtd.resolve(data, response, context);
}).fail(function (errorCode, response, context) {
    that.response = response;
    dtd.reject(errorCode, response, context);
})['catch'](dtd['throw']);
```



### cookie

#### 功能

cookie提供了一套标准的浏览器cookie操作接口，可以安全的进行cookie的查询，创建和删除。通过Simple.Tools.cookie访问，同时兼容浏览器和服务端。如只在node环境使用，可访问  Simple.Server.Tools.cookie。

#### 接口

| 名称    | 输入                                              | 功能                                     |
| ------- | ------------------------------------------------- | ---------------------------------------- |
| getData | name: cookie名, context                           | 获取cookie，可同时用于浏览器端和服务段。 |
| setData | name: cookie名, value: cookie值, options, context | 设置cookie，可同时用于浏览器端和服务段。 |
| remove  | name: cookie名, options, context                  | 移除cookie，可同时用于浏览器端和服务段。 |

### logger

#### 功能

logger提供一套日志记录接口，支持node和浏览器环境。可通过 Simple.Tools.logger 访问。如只在node环境使用，可访问 Simple.Server.Tools.logger。浏览器环境中，如果打开了项目的Simple.config的develop模式，将会在DevTools的console中打印日志。node环境中则在服务器控制台打印日志。

| 名称    | 输入     | 功能                                                  |
| ------- | -------- | ----------------------------------------------------- |
| log     | string   | 打印文本日志。                                        |
| error   | string   | 打印错误日志，控制台中会高亮显示。                    |
| request | response | 打印请求日志，自动组装出请求响应的时间，URL和返回值。 |



## Class

### 功能

Simple5 框架内部实现了一套 Class 机制，没有直接使用 ES6 的 Class 模型以兼容仅支持 ES5 的浏览器环境。系统中所有类型的基类为 SimpleObject。Class机制本质上是通过构造函数和 prototype 实现的，每个类都是一个构造函数，但不要直接调用构造函数，通过Class.create() 方法来实例化以得到完整的属性继承。

### 示例

```javascript
// 定义类
module.exports = SimpleObject.extend({})

// 继承
module.exports = Simple.Store.extend({
    type: 'inline',
    data: [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
})

// 实例化
Application.create({
    params: params,
    context: context
});

// 设置类名
Application.setName(className)
```



## Deferred

### 功能

Deferred 是 Simple5 内部实现的一套异步处理机制，与ES6中的 promise 类似。Deferred 内部有三种状态：pending, resolve, reject，同样是单向状态更新。可以实例化 Deferred 对象后可进行链式操作。

### 接口

| 名称                 | 输入                         | 功能                                                         |
| -------------------- | ---------------------------- | ------------------------------------------------------------ |
| Simple.Deferred.when | 需要封装的操作，支持无限参数 | 对输入的操作进行异步封装，执行完成后调用对应的 done 或 fail 回调。 |
| Simple.Deferred.all  | 需要封装的操作，支持无限参数 | 等所有操作都成功后，将操作结果封装成数组后调用 done 回调。如果有任何操作失败则直接执行 fail 回调。 |
| done                 | function                     | 注册成功回调函数。                                           |
| fail                 | function                     | 注册失败回调函数。                                           |
| then                 | function, function           | 同时注册成功和失败回调函数，第一个参数为成功回调，第二个为失败回调。 |
| always               | function                     | 终章回调函数。                                               |

### 示例

```javascript
// 直接使用 Deferred 进行异步处理
Simple.Deferred.when(
    ListStore.create().request({}),
    ListStore.create().request({}),
    ListStore.create().request({})
).done(function(data1, data2, data3) {
}).fail(function() {
});

// 实例化 Deferred 进行链式操作
var deferred = Simple.Deferred; 
var dtd = deferred();
deferred.when(
    ListStore.create().request({}),
    ListStore.create().request({})
).done(function(data1, data2){
  dtd.resovle(); // 不执行此句会超时
}).fail(function(data1, data2){
});
return dtd;
```



## Message

```javascript
/*
  浏览器tab之间的数据传递，只能传递文本
*/
// A tab 代码
Simple.Message.on('msg',function(data){
  console.log(data);
},true);
// B tab 代码
Simple.Message.emit('msg','text1');
```



## Queue

