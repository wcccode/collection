#<a href='https://github.com/senchalabs/connect'>connect</a> 源码学习
这里主要说下connect的路由拦截处理。connect可以添加多个中间件或路由对http的请求进行处理。
在处理http请求时，connect先获取相应的path，然后遍历相应的路由stack，每次从栈顶开始，
当一个路由和path匹配时，若path符合/path、/path/*、/path.*则使用该路由处理请求，否则都跳过当前路由
继续遍历下一个路由。若路由处理请求成功则终止当前路由遍历，若失败则跳过路由继续路由遍历。
路由的匹配规则：
  1. route.length={0,m}, path.length={0,n}
  2. 当m>n时route != path 跳过该路由，继续遍历
  3. 当m<=n route != path.substring(0, m) 跳过该路由，继续遍历
  4. 当m<=n route = path.substring(0, m) 
  5. path[m]不等于undefined、'/'、'.'时跳过路由，继续遍历，如path=/test1，route=/test
  6. 路由匹配，进行处理，若出现异常则继续遍历

以下是修改注释后的源码
```
var finalhandler = require('finalhandler');
var http = require('http');
var debug = require('debug')('connect:dispatcher');
var parseUrl = require('parseurl');

var app = module.exports = {};

var env = process.env.NODE_ENV || 'development';

//添加中间件或者route处理http的请求
app.use = function(route, fn){
  // default route to '/'
  if ('string' != typeof route) {
    fn = route;
    route = '/';
  }
  
  // wrap sub-apps
  if ('function' == typeof fn.handle) {
    var server = fn;
    server.route = route;
    fn = function(req, res, next){
      server.handle(req, res, next);
    };
  }

  // wrap vanilla http.Servers
  if (fn instanceof http.Server) {
    fn = fn.listeners('request')[0];
  }

  //统一过滤route最后面的'/'  如/test、/test/是一样的
  if ('/' == route[route.length - 1]) {
    route = route.slice(0, -1);
  }

  //路由和对应的处理函数保存在栈里面
  debug('use %s %s', route || '/', fn.name || 'anonymous');
  this.stack.push({ route: route, handle: fn });

  return this;
};

//这里负责处理每个http的请求
app.handle = function(req, res, out) {
  var stack = this.stack
    , search = 1 + req.url.indexOf('?')
    , pathlength = search ? search - 1 : req.url.length
    , fqdn = 1 + req.url.substr(0, pathlength).indexOf('://')
    , protohost = fqdn ? req.url.substr(0, req.url.indexOf('/', 2 + fqdn)) : ''
    , removed = ''
    , slashAdded = false
    , index = 0;

  // final function handler
  var done = out || finalhandler(req, res, {
    env: env,
    onerror: logerror
  });

  // store the original URL
  req.originalUrl = req.originalUrl || req.url;
  //搜索路由对应的处理函数
  function next(err) {
    if (slashAdded) {
      req.url = req.url.substr(1);
      slashAdded = false;
    }

    if (removed.length !== 0) {
      req.url = protohost + removed + req.url.substr(protohost.length);
      removed = '';
    }

    // next callback
    var layer = stack[index++];

    // all done
    if (!layer) {
      done(err);
      return;
    }

    // route data
    var path = parseUrl(req).pathname || '/';
    var route = layer.route;

    // 跳过不匹配的路由，此处会以此时route的长度n截取path的0到n的内容并看是否匹配，
    // 若不匹配则跳过继续下一个路由。若匹配则继续，暂时不管path的大于n后的内容
    if (path.toLowerCase().substr(0, route.length) !== route.toLowerCase()) {
      return next(err);
    }
    
    // 此时有两种情况，一种是/route、/route/* 或 /route.*，另一种是/route*
    // 若第一种情况则继续执行，否则跳过路由
    var c = path[route.length];
    if (c !== undefined && '/' !== c && '.' !== c) {
      return next(err);
    }

    // trim off the part of the url that matches the route
    if (route.length !== 0 && route !== '/') {
      removed = route;
      req.url = protohost + req.url.substr(protohost.length + removed.length);

      // ensure leading slash
      if (!fqdn && req.url[0] !== '/') {
        req.url = '/' + req.url;
        slashAdded = true;
      }
    }
    // 处理相应的路由，若出错则跳过路由继续
    call(layer.handle, route, err, req, res, next);
  }

  next();
};

app.listen = function(){
  var server = http.createServer(this);
  return server.listen.apply(server, arguments);
};

function call(handle, route, err, req, res, next) {
  var arity = handle.length;
  var hasError = Boolean(err);

  debug('%s %s : %s', handle.name || '<anonymous>', route, req.originalUrl);

  try {
    if (hasError && arity === 4) {
      // error-handling middleware
      handle(err, req, res, next);
      return;
    } else if (!hasError && arity < 4) {
      // request-handling middleware
      handle(req, res, next);
      return;
    }
  } catch (e) {
    // reset the error
    err = e;
  }

  // continue
  next(err);
}

function logerror(err){
  if (env !== 'test') console.error(err.stack || err.toString());
}
```
