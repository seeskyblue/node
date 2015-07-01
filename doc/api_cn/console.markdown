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

Uses `util.inspect` on `obj` and prints resulting string to stdout. This function
bypasses any custom `inspect()` function on `obj`. An optional *options* object
may be passed that alters certain aspects of the formatted string:

使用 `util.inspect` 处理 `obj` 并将结果输出到标准输出。该函数会忽略任何 `obj` 上自定义的 `inspect()` 函数。

- `showHidden` - if `true` then the object's non-enumerable properties will be
shown too. Defaults to `false`.

- `depth` - tells `inspect` how many times to recurse while formatting the
object. This is useful for inspecting large complicated objects. Defaults to
`2`. To make it recurse indefinitely pass `null`.

- `colors` - if `true`, then the output will be styled with ANSI color codes.
Defaults to `false`. Colors are customizable, see below.

## console.time(label)

Mark a time.

## console.timeEnd(label)

Finish timer, record output. Example:

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');
    // prints 100-elements: 262ms

## console.trace(message[, ...])

Print to stderr `'Trace :'`, followed by the formatted message and stack trace
to the current position.

## console.assert(value[, message][, ...])

Similar to [assert.ok()][], but the error message is formatted as
`util.format(message...)`.

[assert.ok()]: assert.html#assert_assert_value_message_assert_ok_value_message
[util.format()]: util.html#util_util_format_format
