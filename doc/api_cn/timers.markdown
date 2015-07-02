# Timers

    稳定度: 5 - 锁定

所有的计时器函数都是全局的。不需要使用 `require()` 引用该模块来使用。

## setTimeout(callback, delay[, arg][, ...])

用以按计划在 `delay` 毫秒之后来执行一次性的 `callback` 函数。 返回一个 `timeoutObject` 句柄以便可能使用 `clearTimeout()` 调用。可选的，你也可以传递参数给 `callback` 回调函数。

有必要注意，回调函数可能不是在很精确的 `delay` 毫秒之后被调用 —— Node.js不保证回调函数在精确的时间被调用，也不保证按顺序调用。回调函数会被尽可能的按照指定时间被调用。

## clearTimeout(timeoutObject)

阻止一个 timeout 计时器被触发（调用回调函数）。

## setInterval(callback, delay[, arg][, ...])

To schedule the repeated execution of `callback` every `delay` milliseconds.
Returns a `intervalObject` for possible use with `clearInterval()`. Optionally
you can also pass arguments to the callback.

## clearInterval(intervalObject)

Stops an interval from triggering.

## unref()

The opaque value returned by `setTimeout` and `setInterval` also has the method
`timer.unref()` which will allow you to create a timer that is active but if
it is the only item left in the event loop won't keep the program running.
If the timer is already `unref`d calling `unref` again will have no effect.

In the case of `setTimeout` when you `unref` you create a separate timer that
will wakeup the event loop, creating too many of these may adversely effect
event loop performance -- use wisely.

## ref()

If you had previously `unref()`d a timer you can call `ref()` to explicitly
request the timer hold the program open. If the timer is already `ref`d calling
`ref` again will have no effect.

## setImmediate(callback[, arg][, ...])

To schedule the "immediate" execution of `callback` after I/O events
callbacks and before `setTimeout` and `setInterval` . Returns an
`immediateObject` for possible use with `clearImmediate()`. Optionally you
can also pass arguments to the callback.

Callbacks for immediates are queued in the order in which they were created.
The entire callback queue is processed every event loop iteration. If you queue
an immediate from inside an executing callback, that immediate won't fire
until the next event loop iteration.

## clearImmediate(immediateObject)

Stops an immediate from triggering.
