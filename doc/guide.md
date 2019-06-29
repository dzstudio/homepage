## 安装

使用 Simple5 框架需要确保你的电脑上已经安装了Node.js和npm管理器。由于公司的npm框架都托管在内部npm服务器中，使用是需要设置npm代理。打开CMD或终端执行一下指令：

```javascript
npm set registry http://npm.mucang.cn
```

运行成功后，再输入： npm login，用你的木仓账号密码登录npm私服。账号不要带邮箱后缀，如果你的账号是HR最近注册的，使用HR提供的原始密码。浏览器访问 npm.mucang.cn ，使用木仓账号登录有可查看全部npm框架。

打开CMD或终端执行一下指令安装Simple5框架：

```javascript
npm install -g @framework/simple-cli
```

至此，Simple5 框架已经安装至你的电脑，并且附带一个完整项目生成脚手架：simple-cli.cmd

## 创建你的第一个项目

### 步骤

打开CMD或终端，进入你要创建项目的目录（手动新建一个项目目录），执行指令：simple-cli.cmd

1. 出现 “项目名称:”，输入一个项目名称并回车，自己定义

2. 出现 “是否生成 NodeJs 项目？”，可自行选择输入y或n，y表示项目会自带node.js服务器脚本并采用服务端渲染的方式加载。n表示项目为纯浏览器渲染。

3. 出现 “是否自动下载依赖？”，输入y并回车，此时会自动下载全部npm依赖，也可选择n，稍后通过 npm install 指令安装。

4. 运行 npm run build 对项目进行打包，此时会生成一个新的build目录。使用live-server或其他工具创建静态服务器，可直接访问build目录中打包好的内容。对于服务端渲染的项目（即以上第二步选择y），可直接在根目录下执行 node .\server\main.js 指令启动node服务器，浏览器访问 localhost:2190 即可访问项目。

   ![](https://images-cdn.shimo.im/sPZJtNyoyko05Z0m/image.png!thumbnail)

至此你已经成功创建一个新项目，以下是项目的基本信息：

### package.json

```javascript
{
    "name": "mcw",
    "version": "3.5.0",
    "description": "",
    // 包依赖
    "dependencies": {
        "@framework/simple5": "2.6.3", // 版本号
        "@framework/simple-node": "1.8.3",
        "@framework/simple-gulp": "0.3.7",
        "@base/node-cache": "latest", // 最新版本
        "gulp": "^3.9.1"
    },
    "eslintConfig": {
        "extends": "@base/eslint-config-mucang-base"
    },
    "engines": {
        "node": ">=7.0.0",
        "npm": ">=3.0.0"
    },
    "private": true,
    "simple": {
        "name": "mcw",
        // 运行模式是否为开发模式
        "develop": true,
        "packages": {
            "mcw": "/mcw/",
            "common": "/common/",
            "static": "/static/",
            "simple": "@framework/simple5/src/"
        },
        "dir": {
            "src": "core/src/",
            "build": "core/build/",
            "disk": "core/disk/"
        },
        "root": "/",
        "index": "/index.html",
        "initialize": "mcw:initialize/main",
        "router": "mcw:router/main",
        "server": {
            // 路径指定，编译后，文件依赖的路径会与此保持一致
            "assets": {
                "core": "/core-assets/",
                "index": "/web-assets/"
            },
            // node端ajax超时时间
            "timeout": 10000,
            // node热重启
            "hotDev": true,
            // 网站端口
            "port": 8080,
            // 启动进程数，为true时会自动获取
            "cluster": 4,
            // 错误日志和请求日志路径
            "log": {
                "dir": "../logs/"
            }
        },
        "bale": {
            "zip": true,
            "dir": "../",
            "server": "./server/"
        },
        "compile": {
            "server": [
                "server"
            ],
            "static": [
                "static"
            ],
            "template": {
                "version": "1.0"
            }
        }
    }
}
```

### gulpfile.js

```javascript
即时编译（watch），
项目编译（build），
编译压缩(compile),
编译打包（bale）
```

### server/main.js

```javascript
// node服务启动
Simple.Server.initialize({
    root: __dirname
}, function (server) {

    server.use(function (request, response, next) {
        next()
    }).start().end();
});

```

### 目录结构

```javascript
core // 代码目录
  src  // 源码目录
    mcw 
      application //页面
      component //组件
      initialize //初始化相关，会在页面/组件之前运行
      store // 数据接口
      router // 路由
      common // 公共资源
    static         
    index.html
  build  // 运行目录，develop为true时运行此目录（双端）
  dist  // 编译后目录，develop为false时运行此目（browser）
```

## 

## 新建页面

若要新建页面，你需要在core/src/{项目名}/application目录中新建目录，以下以新建 “home” 为例：

```javascript
/*
  在application目录中，新建一个home目录
*/
// main.js  代码如下
var Server = require('./server/main');
var View = require('./view/main');

module.exports = Simple.Application.extend({
    $constructor: function () {
        var that = this;

        that.props = {};

        that.$super();
    },
    willMount: function () {

    },
    didMount: function (refDom) {
        
    },
    server: Server,
    view: View,
    name: module.id
});

// server/main.js  代码如下

module.exports = function (body) {

    var that = this;
    var opt = {
        title: '主页 - 买车网',
        keywords: '',
        description: ''
    };
    
    that.context.send({
        code: 200,
        type: 'html',
        context: {
            title: opt.title,
            meta: [
                {
                    name: 'keywords',
                    content: opt.keywords
                },
                {
                    name: 'description',
                    content: opt.description
                }
            ],
            name: that.name,
            body: body
        }
    });
};

// view/main.html 代码如下

<meta name="style" content="./main"/>
<div class="home-container">
    欢迎来到主页
</div>

/* view/main.less 代码如下 */
@import "../../../../common/resources/css/common";

.home-container {
  p {
    
  }
}

/*
  在router目录中，新建一个home目录
  main.js 代码如下
*/
var Router = Simple.Router;
var app = require('mcw:application/home/main');

Router.use(['/'], function (params, context) {

    return app.create({
        context: context,
        params: params
    }).render();
});

// router/main.js 文件增加如下代码
require('./home/main');
```



## 使用组件

组件使用 Simple.Component.extend() 函数来创建，组件的生命周期及属性定义如下：

```javascript
var View = require('./view/main');

module.exports = Simple.Component.extend({
    // 组件初始化的时候执行
    $constructor: function () {

        var that = this;
        // 执行父级构造函数
        that.$super();
        /*
          可用来推送刷新DOM
          例如，that.setState({title:'标题1'});
          运行后view中使用该变量的位置数据会刷新
        */ 
        that.state = {

        };
        /*
          属性对象，会接收来自其他组件的传值
          例如，在view中 {{com User {name:4} /com}}
          在User组件中，可以通过that.props.name 获取值
        */
        that.props = {
          
        };

    },
    // 双端都会执行，一般用来执行一些Deffered或者Store
    willMount: function(){
      
    },
    // 仅浏览器端执行
    didMount: function () {
        var that = this;
    },
    // 组件名
    name: module.id,

    // 指定view
    view: View

});

module.exports.setName(module.id);

```

### on、emit

```javascript
/*
  组件CarList中监听
*/
$constructor: function () {
  var that = this;
  
  that.on('count', function(data){
    console.log(data); // 100
  });
}
/*
  其他组件中触发
*/
didMount: function(){
  var that = this;
  
  that.event.on('btn','click',function(){
    that.children.CarList.emit('count',100);
  });  
}
```

### setState()、forceUpdate()

```javascript
/*
1、如果数title值发生了改变，则刷新dom数据。
2、使用此方法时，在该组件下会清除掉所有使用js操作dom的行为
3、参数2表示是否同步
*/
that.setState({
  title: '标题1'
});

// setProps 作用同 setState一样
// that.forceUpdate()
```

### 

## 网络请求

### 创建Simple.Store对象

Simple5框架默认使用Simple.Store类型来处理网络请求，其内部已封装了兼容浏览器和node的网络调用机制。

```javascript
module.exports = Simple.Store.extend({
    /*
      online时会发出请求，执行dataFilter，并填充data
      inline时会直接使用data，不执行dataFilter，使用Mock数据调试时使用inline模式
    */
    type: 'online', 
    // 类型为json,text，默认json
    dataType: 'json',
    // 指定为jsonp方式调用，默认false
    jsonp: true,
    // type=inline时需要设置
    data: [],
    // 请求参数
    params: {},
    // POST|GET，默认get
    method: 'GET',
    // 请求地址
    url: 'mcw://api/open/price/get-car-range-price.htm',
    // 请求完成后，数据处理
    dataFilter: function(type, options, json){
      
      return json;
    },
    // 请求之前，数据处理
    before: function(dtd, params){
      dtd.resolve([{a:1}]);
      
      return false;
    }
});

```

### 发出请求

```javascript
var Store = require('./store/main');

Store.create({
  context: that.context
}).request(params).done(function(data){
  
})

```



## 使用路由

router/main.js 该文件中存放页面路由配置，事实上该文件为项目真正的入口文件。项目启动后会加载该文件，进行路由匹配后，执行对应的路由回调，最终通过 render() 函数渲染页面。路由的具体配置如下：

```javascript
var Router = Simple.Router; // 获取路由处理模块
// 通过Router.use注册路由，第一个参数为网址路由规则数组，以下表示匹配 "/" 和 "/index.html" 两个路径。
// 第二个参数为路由处理回调，匹配成功后执行。
// 第三个参数为自定义参数，传入json对象。该参数会继续传入路由处理回调中。
Router.use(['/', './index.html'], function (params, context) {
    // 加载'application/home/main'模块，并创建实例化该模块，传入请求参数和上下文。
    return require.async('pingxing:application/home/main', function (app) {
        app.create({
            params: params, // 请求参数
            context: context// 上下文，node模式下，通过上下文向浏览器输出页面html
        }).render();
    });
});
```



## 模板渲染

以 home/view/main.html 为例，该文件虽然是.html文件，仍可以使用 var View = require('./view/main'); 进行加载。构建时Simple5会自动将器解析成Simple.View类型的 js 模块。同大多数模板一样，Simple.View中可以直接在模板中使用 变量输出，条件语句，循环，组件引入等功能：

### 使用self对象

```javascript
<script>
  // 在模板中，self只想组件本身
  var data = self.props.data;
</script>
```

### style

```javascript
<!-- 仅在name=style时，才为引入样式 -->
<meta name="style" content="./main">
```

### script

```javascript
<script>
  // 模板中可新建script标签编写javascript代码。
  var props = self.props;
</script>
<div>
  {{=props.name}}
</div>
```

### each

```javascript
<!--
  $index 索引
  $value 值
  each 可以遍历对象
  as 用b 和 i 取代 $index 和 $value
-->
<ul>
   {{each arr}}
      <li>               
         {{=$index}}-{{=$value.title}}
      </li>
    {{/each}}
</ul>

<ul>    
{{each arr as b i}}       
  <li>                        
    {{=i}}-{{=b.title}}       
  </li>     
{{/each}} 
</ul>

```

### if

```javascript
{{if data === 1}}
<div class="tip">1</div>
{{else if data === 2}}
<div class="tip">2</div>
{{else}}
<div class="tip">2</div>
{{/if}}
```

### com 引入组件

```javascript
<meta name="List" content="mcw://application/header/com/list/main">
<meta name="Footer" content="../../com/footer/main">

<script>
  // 组件传值，会注入到该组件的props属性中
  var data = {
    name: '奔驰'
  };
</script>
<div class="style1">
  {{com List data /com}}
  {{com Footer /com}}
</div>

```

### include 引入html页

```javascript
<script>
  var data = self.props.info;
</script>
<div class="style1">
  {{include './right' data }}
  {{include './content' }}
</div>
```

### = 输出值

```javascript
{{=props.name}}
```

### =# 输出带HTML格式的值

```javascript
{{=#props.content}}
```

### {{}} 脚本块

```javascript
<!-- 会在node和浏览器控制台打印数据 -->
<div class="style1">
  {{console.log(data)}} 
</div>
```

## 

## DOM和事件

### event

```javascript
var that = this;
/*
  参数1：ref名称
  参数2：事件
  e.refTarget: 当前DOM对象，e与传统事件中的event一样
*/
that.event.on('tab', 'mouseover', function (e) {
    that.setState({
        show: parseInt(e.refTarget.getAttribute('data-index'))
    });
});
// 解绑事件
that.event.off('ref', 'mouseover');
// 触发事件
that.event.trigger('ref', 'mouseover');

```

### refDOM

```javascript
﻿// HTML 部分 
<div ref="name">123</div> 
// didMount 部分 
didMount: function(refDom){
  var sel = refDom.name; 
  // 找到1个返回div element，多个则返回数组
}
﻿﻿﻿﻿﻿﻿﻿
```

### getDOMNode()

```javascript
// HTML 部分
<div ref="name">123</div>
// didMount 部分
var sel = that.getDOMNode()['name找到1个']// 返回以上div ，多个则返回数组element

```

