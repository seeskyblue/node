# console

    稳定度: 4 - API冻结

* {Object}

<!--type=global-->

用于输出标准输出（stdout）和标准错误输出（stderr）。类似于大多说浏览器提供的控制台对象方法，但在Node中输出被发送到标准输出和标准错误输出。

当目标是终端或文件的时候，控制台方法是同步的（以避免过早退出时丢失信息）。而当目标是管道时，是异步的（以避免长时间的阻塞）。

在下面的例子中，标准输出是非阻塞的，而标准错误输出是阻塞的：

    $ node script.js 2> error.log | tee info.log

在日常使用中，阻塞还是非阻塞并不是你所需要担心的，除非你的日志包含非常大量的数据需要输出。

## console.log([data][, ...])

在标准输出中输出新行。该函数可以接受多个参数，类似于 `printf()` ，例如：

    var count = 5;
    console.log('count: %d', count);
    // prints 'count: 5'

如果格式化元素没有在第一个字符串中找到，那么 `util.inspect` 便会用来处理每一个参数。
更多信息参见 [util.format()][] .

## console.info([data][, ...])

类似 `console.log` 。

## console.error([data][, ...])

类似 `console.log` ，但是输出到标准错误输出（stderr）。

## console.warn([data][, ...])

类似 `console.error` 。

## console.dir(obj[, options])

使用 `util.inspect` 处理 `obj` 并将结果字符串输出到标准输出。该函数会忽略任何 `obj` 上自定义的 `inspect()` 函数。可选参数 *options* 对象可以传入来改变格式化字符串的某些具体方面：

- `showHidden` - 当为 `true` 时，对象的不可枚举（non-enumerable）属性也会被显示出来，默认为 `false` 。

- `depth` - 告诉 `inspect` 在格式化对象时进行多少次递归调用。这对检查大型复杂的对象时非常有用。默认为 `2` 。传入 `null` 进行无限次数的递归调用。

- `colors` - 当为 `true` 时，输出结果的样式会根据美国国家标准学会（ANSI）标准来着色代码。默认为 `false` 。颜色是可以自定义的，参见后续介绍。

## console.time(label)

标记一个时间。

## console.timeEnd(label)

结束计时器，记录输出结果。例如：

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');
    // prints 100-elements: 262ms

## console.trace(message[, ...])

输出到标准错误输出 `'Trace :'` ，后面紧接着格式化的消息字符串和当前代码位置的堆栈调用信息。

## console.assert(value[, message][, ...])

类似 [assert.ok()][], 但是错误信息是经过格式化的，如同 `util.format(message...)`。

[assert.ok()]: assert.html#assert_assert_value_message_assert_ok_value_message
[util.format()]: util.html#util_util_format_format
