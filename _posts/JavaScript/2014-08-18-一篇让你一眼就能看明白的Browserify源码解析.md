## Browserify 生成源码分析

今天贴上一篇一眼就能看明白的`browserify`生成的源码分析.

---

### 一个编译还原并附带注释的源码分析

```js

(function createRequire(modules, caches, defaults) {
    function factory(index, u) {
        // 如果模块没加载过
        if (!caches[index]) {
            // 如果模块在模块列表里未定义
            if (!modules[index]) {
                // 如果全局环境存在require函数定义
                var a = typeof require == "function" && require;
                if (!u && a)
                    // 直接调用全局环境的require来请求模板
                    return a(index, !0);
                if (i)
                    // 直接调用全局环境的require来请求模板
                    return i(index, !0);
                throw new Error("Cannot find module '" + index + "'")
            }
            // 向caches模块对象库里定义当前对象
            var module = caches[index] = {exports: {}};
            // 请求模块列表属性数组第一项源码
            modules[index][0].call(module.exports, function(path) {
                // path 代表require里的参数 即模块路径
                // 假如源码里依赖别的模块，则调用数组第二项数据
                var depIndex = modules[index][1][path];
                // 进入加载依赖阶段
                return factory(depIndex ? depIndex : path)
            }, module, module.exports, createRequire, modules, caches, defaults)
        }
        return caches[index].exports
    }
    var i = typeof require == "function" && require;
    // 初始化会走这里
    for (var _index = 0; _index < defaults.length; _index++)
        factory(defaults[_index]);
    // 返回一个模块工厂函数
    return factory;
})(
    // 下面是分析完之后的模块列表
    // 属性数组里,第一项为包装好的源码,第二项为依赖的列表索引
    // 后面依次
    {
        1:  [
                function(require, module, exports) {
                    var app = require('./module/app.js');
                    document.querySelector('p').innerHTML = app.get();
                }, 
                {"./module/app.js": 2}
            ],
        2:  [
                function(require, module, exports) {
                    var app = {
                        get: function() {
                            return 'feenan!';
                        }
                    }
                    module.exports = app;
                }, 
                {}
            ]
    }, 
    // 此对象保存已经请求的对象
    {}, 
    // 默认请求第一个
    [1]
)

```

