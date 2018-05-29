
## 流程

Koa输出Application，继承于Emitter，

包含属性：

proxy 代理

middleware 中间件数组

subdomainOffset

env 环境


context  主体

request 

response

方法：

lisiten(...args)：监听函数，主要执行http.createServer(this.callback())，并传入回调函数

toJSON()：返回3个参数：proxy，subdomainOffset，env

inspect()：返回toJSON

use(fn):如果fn不是函数，返回错误，如果是generation函数,转换成convert函数，最终将函数push进middleware数组


code1:
```js

  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    if (isGeneratorFunction(fn)) {
      fn = convert(fn);
    }
    this.middleware.push(fn);
    return this;
  }

```


code convert:
```js

function * legacyMiddleware (next) {
  // before
  yield next
  // after
}
 
function modernMiddleware (ctx, next) {
  // before
  return next().then(() => {
    // after
  })
}

```

callback():
1. 挂载中间件，类似while循环转换middleware中的中间函数为dispatch()，next()即执行下一个中间件函数，如果是数组中最后一个，返回一个promise.resolve()命名为fn


中间件函数：
```js
function (context, next) {
   //  let index = -1 防止重复执行
    function dispatch (i) {
    //  if (i <= index) return Promise.reject(new Error('next() called multiple times'))
    //  index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }

    return dispatch(0)
    

  }

```


2. 创建一个ctx，在this.context的基础上，加入其它属性包括：
request、response、app (this)、req、res、originalUrl、cookies、ip

3. 最终将ctx和fn作为参数交给handleRequest函数




handleRequest(ctx,fn):定义res.statusCode，初始值404，注册errorHandle；responseHandle，然后执行中间件函数，如果都resolve了，handle response，如果reject，handle error


responseHandle(ctx):把ctx.body交给ctx.res（http的response）



## context

方法：

toJSON()：返回request、response、app (this)、req、res、originalUrl、socket

onerror():    this.app.emit('error', err, this);然后设置请求头，code，msg，status，然后res出来





delegate

```

delegate(proto, 'response')
  .method('attachment')
  .method('redirect')
  .method('remove')
  .method('vary')
  .method('set')
  .method('append')
  .method('flushHeaders')
  .access('status')
  .access('message')
  .access('body')
  .access('length')
  .access('type')
  .access('lastModified')
  .access('etag')
  .getter('headerSent')
  .getter('writable');

```


```
Delegator.prototype.method = function(name){
  var proto = this.proto;
  var target = this.target;
  this.methods.push(name);

  proto[name] = function(){
    return this[target][name].apply(this[target], arguments);
  };

  return this;
};

```

proto.redirect = ()=>{
    this['response']['redirect'].apply(this['response'],arguments)
}

执行proto.redirect相当于执行proto.response.redirect


## request
## response

request和response主要定义了一些getter和setter方法，封装原生http模块的request和response



## 请求流程


```
graph TD

A[请求]-->B[http接受请求]
B-->D[调用callback,执行中间件]
D-->E[返回响应结果]

```
