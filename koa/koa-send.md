# koa-send 3.1.1

send(ctx,path,opts)将path路径下的文件以流形式发送到客户端。
参数ctx为koa的应用上下文。参数path为文件路径或文件目录，文件目录须opts.index配置。

    var debug = require('debug')('koa-send');
    var resolvePath = require('resolve-path');
    var assert = require('assert');
    var path = require('path');
    var normalize = path.normalize;
    var basename = path.basename;// 返回路径的最后一部分
    var extname = path.extname;// 扩展名
    var resolve = path.resolve;
    var parse = path.parse;// 将路径转化为对象
    var sep = path.sep;// 路径分割符
    var fs = require('mz/fs');
    var co = require('co');
    
    module.exports = send;
    
    // 将文件路径path中的文件以流形式发送到客户端
    function send(ctx, path, opts) {
      return co(function *(){
    
        assert(ctx, 'koa context required');
        assert(path, 'pathname required');
        opts = opts || {};
    
        // options
        debug('send "%s" %j', path, opts);
        var root = opts.root ? normalize(resolve(opts.root)) : '';
        var trailingSlash = '/' == path[path.length - 1];// 是否以"/"结尾
        path = path.substr(parse(path).root.length);// 可接受url形式的文件路径
        var index = opts.index;// 选择文件目录下[index]文件
        var maxage = opts.maxage || opts.maxAge || 0;// 文档被访问后的存活时间
        var hidden = opts.hidden || false;// 是否支持"."起始的文件或文件目录
        var format = opts.format === false ? false : true;// path若为文件目录，是否自动选取其下[index]文件
        var gzip = opts.gzip === false ? false : true;
    
        // ctx.acceptsEncodings校验请求头中'accept-encoding'属性是否匹配传参数组中的任意项；若匹配返回匹配值
        // 参数如['gzip', 'deflate', 'identity']
        var encoding = ctx.acceptsEncodings('gzip', 'deflate', 'identity');
    
        path = decode(path);
    
        if (-1 == path) return ctx.throw('failed to decode', 400);
    
        // 选择path路径下[index]文件
        if (index && trailingSlash) path += index;
    
        path = resolvePath(root, path);
    
        if (!hidden && isHidden(root, path)) return;
    
        if (encoding === 'gzip' && gzip && (yield fs.exists(path + '.gz'))) {
          path = path + '.gz';
          ctx.set('Content-Encoding', 'gzip');
          ctx.res.removeHeader('Content-Length');
        }
    
        // 若为文件目录，选取其下[index]文件
        try {
          var stats = yield fs.stat(path);
    
          if (stats.isDirectory()) {
            if (format && index) {
              path += '/' + index;
              stats = yield fs.stat(path);
            } else {
              return;
            }
          }
        } catch (err) {
          var notfound = ['ENOENT', 'ENAMETOOLONG', 'ENOTDIR'];
          if (~notfound.indexOf(err.code)) return;
          err.status = 500;
          throw err;
        }
    
        ctx.set('Last-Modified', stats.mtime.toUTCString());
        ctx.set('Content-Length', stats.size);
        ctx.set('Cache-Control', 'max-age=' + (maxage / 1000 | 0));
        ctx.type = type(path);// 文件扩展名
        ctx.body = fs.createReadStream(path);// 以流形式设置应用上下文的body属性
    
        return path;
      });
    }
    
    function isHidden(root, path) {
      path = path.substr(root.length).split(sep);
      for(var i = 0; i < path.length; i++) {
        if(path[i][0] === '.') return true;
      }
      return false;
    }
    
    // 获取文件扩展名，先移除压缩文件扩展名".gz"后再获取
    function type(file) {
      return extname(basename(file, '.gz'));
    }
    
    // 特殊字符解码
    function decode(path) {
      try {
        return decodeURIComponent(path);
      } catch (err) {
        return -1;
      }
    }
