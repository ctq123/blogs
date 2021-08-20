# 揭秘webpack的proxy代理日志

今天我们聊一聊webpack代理日志，请大家先思考一个问题，如何设计代理日记？

## 背景
我们日常很多项目通常在本地开发的时候都会代理到不同的环境，而有时候会根据不同的url转发到特定的服务器，当项目复杂度越高的时候，就会出现各种各样的问题，因此代理日志这时就显得尤为重要。

最近在优化webpack的代理日志打印发现了一个问题，有的项目中通常会打印代理请求转发的日志（多为vue项目），而有的项目中却缺少类似的日志（多为react项目），如下：
![图片](https://cdn.poizon.com/node-common/4f5843e3be6ed15c26ed77981e08eb8473d3db24807be86d99b934fe02559f6a.png)

里面包含了请求方式、请求路径、以及转发的地址。尤其是排查问题的时候，有时候你并不能确定这个路由就是转发到对应的域名或者IP上，这时这种类型的日志就大显身手了。

## 分析
开始怀疑是vue-cli内部集成了代理日志的打印，莫非是vue-config.js对代理Proxy进行了内部处理？在proxy做了一层请求拦截并打印相关的日志？

由于vue-cli内部集成的是webpack的配置，但经过搜索vue-cli内部的webpack配置并没有找到相关的源码，因此可以肯定并不是vue-cli处理代理日志。

排除了vue-cli后，回头查看webpack官网，查询打印日志的等级和proxy并没有找到对应的配置。

但从中发现了一个重要的信息，那就是webpack的proxy功能是使用了 [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) 来实现的。于是决定到它的官网上去查找一番，里面的配置项有很多，然而如果你想看代理的日志配置，里面并没有很明显地写出来的。

经过查阅http-proxy-middleware的源码终于发现了它的真身，它是在请求前统一进行了拦截处理，源码如下：
```javascript
...
private prepareProxyRequest = async (req: Request) => {
  // https://github.com/chimurai/http-proxy-middleware/issues/17
  // https://github.com/chimurai/http-proxy-middleware/issues/94
  req.url = req.originalUrl || req.url;

  // store uri before it gets rewritten for logging
  const originalPath = req.url;
  const newProxyOptions = Object.assign({}, this.proxyOptions);

  // Apply in order:
  // 1. option.router
  // 2. option.pathRewrite
  await this.applyRouter(req, newProxyOptions);
  await this.applyPathRewrite(req, this.pathRewriter);

  // debug logging for both http(s) and websockets
  // 关键在这，设置logLevel为debug模式
  if (this.proxyOptions.logLevel === 'debug') {
    const arrow = getArrow(
      originalPath,
      req.url,
      this.proxyOptions.target,
      newProxyOptions.target
    );
    this.logger.debug(
      '[HPM] %s %s %s %s',
      req.method,
      originalPath,
      arrow,
      newProxyOptions.target
    );
  }

  return newProxyOptions;
};
....
```
> Tips: 也可以通过debug webpack的方式来查找，就是配置繁琐一些

由上述源码可知，webpack的proxy处理请求代理转发前，需要做两件事情，一是处理路由，二是处理路径，最后再根据每个路由下的配置logLevel是否为debug模式来决定要不要打印相关的日志。

#### logLevel与clientLogLevel有什么区别？
+ clientLogLevel是相对于webpack全局的，负责全局日志的打印，它的取值为：
```javascript
devServer.clientLogLevel 
string = 'info': 'silent' | 'trace' | 'debug' | 'info' | 'warn' | 'error' | 'none' | 'warning'
```
+ logLevel是对proxy的日志，而且是每个路由下面的配置，负责的是代理路由的日志打印，它的取值为：
```javascript
// log level 'weight'
enum LEVELS {
  debug = 10,
  info = 20,
  warn = 30,
  error = 50,
  silent = 80,
}
```

## 结论
这样问题就清晰了，如果想要查看代理转发的日志我们只需按如下配置即可：
![图片](https://cdn.poizon.com/node-common/135fb584fad94f573be9d200d2fa27835c24f8e6497c0e3f96743cb800057d44.png)

> 注意，如果是做了跨域的代理，必须将changeOrigin设置为true

好了，如果我们想看代理转发的日志，我们只需将logLevel: 'debug'就可以好好的玩耍了，强烈建议webpack的项目设置这一个选项。

