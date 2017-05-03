# koa-is-json 1.0.0

koa-is-json模块用于判断响应是否需要作为json数据处理。

	module.exports = isJSON;
	
	function isJSON(body) {
	  if (!body) return false;
	  if ('string' == typeof body) return false;
	  if ('function' == typeof body.pipe) return false;// 作为流
	  if (Buffer.isBuffer(body)) return false;// 作为buffer对象
	  return true;
	}