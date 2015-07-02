# Timers

    稳定度: 5 - 锁定

所有的定时器函数都是全局的。不需要使用 `require()` 引用该模块来使用。

## setTimeout(callback, delay[, arg][, ...])

用以按计划在 `delay` 毫秒之后单词执行 `callback` 回调函数。 返回一个 `timeoutObject` 句柄以便可以被 `clearTimeout()` 使用。可选的，你也可以传递参数给 `callback` 回调函数。

有必要注意，回调函数可能不是在很精确的 `delay` 毫秒之后被调用 —— Node.js不保证回调函数在精确的时间被调用，也不保证按顺序调用。回调函数会被尽可能的按照指定时间被调用。

## clearTimeout(timeoutObject)

阻止一个 timeout 定时器被触发（调用回调函数）。

## setInterval(callback, delay[, arg][, ...])

用以按计划在每个 `delay` 毫秒之后重复执行 `callback` 回调函数。返回一个 `intervalObject` 句柄以便可以被 `clearInterval()` 使用。可选的，你也可以传递参数给 `callback` 回调函数。

## clearInterval(intervalObject)

阻止一个 interval 定时器被触发（调用回调函数）。

## unref()

由 `setTimeout` 和 `setInterval` 函数调用后返回的句柄也具有 `timer.unref()` 方法，允许你创建一个有效的定时器，但是如果它是（程序）事件循环中仅剩的项目时并不会保持程序运行。如果该定时器已经调用了 `unref` 又再次调用 `unref` 不会有任何效果。

在使用 `setTimeout` 的时候，当你调用 `unref` ，会创建另一个独立的定时器并唤醒事件循环。创建过多类似的定时器会对事件循环性能造成不利影响 —— 请小心使用。

## ref()

如果你之前已经对一个定时器调用了 `unref()` ，你可以调用 `ref()` 来明确要求定时器保持程序运行。如果该定时器已经调用了 `ref` 又再次调用 `ref` 不会有任何效果。

## setImmediate(callback[, arg][, ...])

用以按计划在I/O事件回调函数之后， `setTimeout` 和 `setInterval` 触发之前，“立即”执行 `callback` 回调函数。返回一个 `immediateObject` 句柄以便可以被 `clearImmediate()` 使用。可选的，你也可以传递参数给 `callback` 回调函数。

immediate 定时器的回调函数的执行顺序，是按照 immediate 定时器创建的先后顺序排列执行的。整个回调函数队列是在每个事件循环的迭代被处理执行的。如果你从一个运行时的回调函数中往队列中添加一个 immediate 定时器，那么该 immediate 定时器不会被调用，直到下一次事件循环的迭代。

## clearImmediate(immediateObject)

阻止一个 immediate 定时器被触发（调用回调函数）。
