# 全局对象

<!-- type=misc -->

这些对象在所有模块中都能被访问。有些对象实际上不是在全局范围而只是在模块范围内可访问，这些对象会有提示。

## global

<!-- type=global -->

* {Object} 全局命名空间对象。

在浏览器中，顶级作用域是全局作用域。这意味着在浏览器中，如果你在全局作用域中，`var something` 会定义为一个全局变量。在Node中这是不同的。顶级作用域并非全局作用域，在Node模块中 `var something` 只会成为这个模块的本地变量，即只属于该模块。

## process

<!-- type=global -->

* {Object}

进程对象，参见 [process object][] 章节。

## console

<!-- type=global -->

* {Object}

控制台对象，用以打印标准输出和标准错误输出，参见 [console][] 章节。

## Class: Buffer

<!-- type=global -->

* {Function}

缓存类，用以处理二进制数据，参见 [buffer][] 章节。

## require()

<!-- type=var -->

* {Function}

require()函数，参见 [Modules][] 章节。 `require` 准确地说并不是一个全局方法，而是每个模块自有的本地方法。


### require.resolve()

* {Function}

使用内部 `require()` 机制查找模块的路径，但并不加载模块，只是返回解析过的文件路径。

### require.cache

* {Object}

当模块被加载是缓存在该对象中。通过从该对象中删除对应键值对，下一次调用 `require` 会重新加载这个模块。

### require.extensions

    稳定度: 0 - 不赞成

* {Object}

指示 `require` 如何去处理某些特定的文件后缀名。

例如：处理后缀名为 `.sjs` 的文件如何 `.js` 文件

    require.extensions['.sjs'] = require.extensions['.js'];

**不赞成**  

之前，该列表被用于按需编译非JavaScript的模块并加载到Node中。然而在使用中发现，有许多更好的方法来实现。例如通过其他的Node程序来加载模块，或者提前将他们编译为JavaScript。

由于模块系统（API）已被锁定，该功能可能永远不会去掉。但他可能会产生微妙的错误和复杂的问题，最好留着不再使用。

## __filename

<!-- type=var -->

* {String}

当前被执行代码的文件路径。是该代码经过解析后的绝对路径。对于主程序来说，这和命令行中所用的文件路径未必是相同的。此变量在模块内部的值就是该模块文件的绝对路径。

例如: 从 `/Users/mjr` 调用 `node example.js`

    console.log(__filename);
    // /Users/mjr/example.js

`__filename` 并不是全局变量，而是模块本地变量。

## __dirname

<!-- type=var -->

* {String}

当前执行脚本所在目录的目录名。

例如: 从 `/Users/mjr` 调用 `node example.js`

    console.log(__dirname);
    // /Users/mjr

`__dirname` 并不是全局变量，而是模块本地变量。


## module

<!-- type=var -->

* {Object}

对当前模块的引用。特别的 `module.exports` 被用来定义该模块的输出，并且可以通过 `require()` 获取。

`module` 并不是全局对象，而是模块本地对象。

更多信息，参见 [module system documentation][] 。

## exports

<!-- type=var -->

对 `module.exports` 的引用的简写。
何时使用 `exports` 或 `module.exports` 的详细信息，参见 [module system documentation][] 。

`exports` 并不是全局变量，而是模块本地变量。

更多信息，参见 [module][] 章节。

## setTimeout(cb, ms)

*至少* 在 `ms` 毫秒之后调用回调函数 `cb` 。实际延迟时间依赖于外部因素，例如操作系统的时间颗粒度和系统负载。

`ms` 值必须在1 ~ 2,147,483,647（上下包含）之内。如果值超出该范围，会被调整为1毫秒。概括来讲，时间跨度不能超过24.8天。

返回一个代表该计时器的句柄。

## clearTimeout(t)

停止之前通过 `setTimeout()` 创建的计时器，计时器的回调函数不会再被执行。

## setInterval(cb, ms)

在每个 `ms` 毫秒周期重复调用回调函数 `cb` 。注意实际的调用周期是变化的，依赖于外部因素例如操作系统时间颗粒度和系统负载。绝不会少于 `ms` ，只可能会更久。

周期间隔的值必须在1 ~ 2,147,483,647（上下包含）之内。如果值超出该范围，会被调整为1毫秒。概括来讲，时间跨度不能超过24.8天。

返回一个代表该计时器的句柄。

## clearInterval(t)

停止之前通过 `setInterval()` 创建的计时器，计时器的回调函数不会再被执行。

<!--type=global-->

计时器函数属于全局变量. 参见 [timers][] 章节。

[buffer]: buffer.html
[module]: modules.html
[Modules]: modules.html#modules_modules
[process object]: process.html#process_process
[console]: console.html
[timers]: timers.html
