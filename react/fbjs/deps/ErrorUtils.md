# ErrorUtils

    "use strict";
    
    if (global.ErrorUtils) {
      module.exports = global.ErrorUtils;
    } else {
      var ErrorUtils = {
        applyWithGuard: function applyWithGuard(callback, context, args, onError, name) {
          return callback.apply(context, args);
        },
        guard: function guard(callback, name) {
          return callback;
        }
      };
    
      module.exports = ErrorUtils;
    }