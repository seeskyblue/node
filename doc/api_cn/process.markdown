# process 进程

<!-- type=global -->

`process` 对象是全局对象，可以在任何地方访问到。他是一个 [EventEmitter][] 的实例。

## Exit Codes 退出代码

Node 当没有其他等待的异步操作时，通常会返回退出状态代码 `0`。以下状态代码被用在其他情况：

* `1` **未捕获的致命异常** - 有一个未捕获的异常，并且没有被任何领域或者 `uncaughtException` 事件处理。

* `2` - 未使用 -（为 Bash 保留的内置误用）。

* `3` **内部 JavaScript 解析错误** - JavaScript 代码内部在 Node 引入过程中引起解析错误。这非常罕见，一般只会在开发 Node 自身的时候发生。

* `4` **内部 JavaScript 执行失败** - JavaScript 代码内部在 Node 引入过程中无法返回一个函数值。这非常罕见，一般只会在开发 Node 自身的时候发生。

* `5` **致命错误** - 在 V8 中有致命且无法恢复的错误。通常一个消息会被输出到标准错误输出，并带有 `FATAL ERROR` 前缀。

* `6` **失效函数内部异常处理** - 有一个未捕获的异常，但是内部致命异常的处理函数不知为何失效，并且不能被调用。

* `7` **内部异常处理运行时错误** - 有一个未捕获异常，并且内部致命异常处理函数在试图处理它时自己抛出了错误。这是有可能发生的，例如，当 `process.on('uncaughtException')` 或者 `domain.on('error')` 处理抛出一个错误。

* `8` - 未使用 - 在 Node 之前的版本中，退出代码 8 有时描述为一个未捕获的异常。 

* `9` - **无效参数** - 一个未知的选项被指定，或者选项需要的值未提供。

* `10` **内部 JavaScript 运行时失败** - JavaScript 源代码内部在 Node 引入过程中当引入函数被调用时抛出错误。这非常罕见，一般只会在开发 Node 自身的时候发生。

* `12` **无效调试参数** - `--debug` 和/或 `--debug-brk` 选项被设置，但是选用了无效的端口号。

* `>128` **信号退出** - 当 Node 接收到终止信号，例如 `SIGKILL` 或 `SIGHUP`，那么退出状态代码将会是 `128` 加上该信号代码。这是一个标准 Unix 的实践，由于退出状态代码被设计为7位（二进制）的整数，而终止信号被设在高位，因此这样就能包含信号代码的值。

## Event: 'exit' 事件：'exit'（退出）

当进程将要退出时被触发。在该事件中，无法阻止事件循环的退出。并且一旦所有的 `exit` 监听完成执行后，进程将退出。因此，在处理过程中 **必须** 只能执行 **同步** 操作。这是一个好的时机来执行模块状态的检查（例如单元测试）。该事件回调函数有一个参数，就是进程退出的代码。

监听 `exit` 事件的例子：

    process.on('exit', function(code) {
      // do *NOT* do this
      setTimeout(function() {
        console.log('This will not run');
      }, 0);
      console.log('About to exit with code:', code);
    });


## Event: 'beforeExit' 事件：'beforeExit'（退出之前）

该事件在 Node 清空事件循环并且没有计划的事件时被触发。一般，Node 没有计划任务的时候便会退出。但是监听 'beforeExit' 事件可以产生异步调用，并可以让 Node 继续运行。

当退出是由明确的终止引起时，比如 `process.exit()`，'beforeExit' 事件不会被触发。并且该事件不应该被用为 'exit' 事件的代替，除非想要计划更多的工作。

## Event: 'uncaughtException' 事件：'uncaughtException'（未捕获异常）

当异常一路冒泡返回到事件循环的时候被触发。如果有该异常的监听被添加，默认行为（打印堆栈跟踪信息并退出）将不会发生。

监听 `uncaughtException` 事件的例子：

    process.on('uncaughtException', function(err) {
      console.log('Caught exception: ' + err);
    });

    setTimeout(function() {
      console.log('This will still run.');
    }, 500);

    // Intentionally cause an exception, but don't catch it.
    nonexistentFunc();
    console.log('This will not run.');

注意，`uncaughtException` 事件是一个非常原始的异常处理的机制。

不要使用它，用 [domains](domain.html) 代替。如果你真的要使用它，每次遇到未处理异常之后要重启应用！

*不要* 使用该事件，觉得 Node.js 就像是 `遇到错误就重新开始下一次`。一个未处理的异常意味着你的应用——引申开来也是 Node.js 自身——处在一个未定义的情况下。盲目的重新开始意味着 *任何事情* 可能会发生。

想象一下重新开始就像你在升级你的系统时拔掉了电源线。十次里面有九次什么事也没有发生——但是第10次的时候，你的系统毁坏了。

你已经被警告过了。

## Signal Events 信号事件

<!--type=event-->
<!--name=SIGINT, SIGHUP, etc.-->

当进程接收到一个信号时被触发。可以参见 sigaction(2)，标准可移植性操作系统接口（POSIX）的信号名称列表，例如SIGINT、SIGHUP等。

监听 `SIGINT` 的例子：

    // Start reading from stdin so we don't exit.
    process.stdin.resume();

    process.on('SIGINT', function() {
      console.log('Got SIGINT.  Press Control-D to exit.');
    });

一个发送 `SIGINT` 简单的方法是 `Ctrl-C`，在大多数的终端中都有效。

注：

- `SIGUSR1` 被 Node.js 保留以启动调试器。可以创建监听器监听，但并不会阻止调试器启动。

- `SIGTERM` 和 `SIGINT` 在非 Windows 平台上有默认处理，即在退出状态代码为 `128 + 信号代码` 时重置终端模式。当这些信号有创建监听时，它们的默认行为将被取消。
  （Node 将不会退出）。
  
- `SIGPIPE` 默认被忽略，可以创建监听。

- `SIGHUP` 在 Windows 平台上当控制台窗口被关闭时产生，其他平台上各种类似的情况下，参见 signal(7)。可以创建监听，但是 Node 还是会被 Windows 无条件的在大约10秒后终止。在非 Windows 平台，`SIGHUP` 的默认行为是终止 Node，但一旦监听被创建，它的默认行为就会被取消。

- `SIGTERM` 在 Windows 平台不被支持，可以创建监听。

- `SIGINT` 由终端产生，可以被所有平台支持。通常可以由 `CTRL+C` 产生（不过这是可设置的）。当终端的 raw 模式启动时不会产生。

- `SIGBREAK` 在 Windows 平台当按下 `CTRL+BREAK` 时释放，在非 Windows 平台可以被监听，但是没有方法可以发送或产生该信号

- `SIGWINCH` is delivered when the console has been resized. On Windows, this will
  only happen on write to the console when the cursor is being moved, or when a
  readable tty is used in raw mode.

- `SIGWINCH` 当控制台窗口被调整大小时释放。在 Windows 平台上只会在光标正在被移动时写入控制台时发生，或当一个可读的 tty 被用在 raw 模式时。

- `SIGKILL` 无法创建监听，它会在所有平台上无条件终止 Node。

- `SIGSTOP` 无法创建监听。

注，Windows 平台不支持发送信号，但是 Node 提供了一些模拟方法，比如 `process.kill()` 和 `child_process.kill()`：

- 发送信号 `0` 可以被用来寻找进程的存在。

- 发送 `SIGINT`、 `SIGTERM` 和 `SIGKILL` 会引起目标进程无条件地退出。

## process.stdout

一个指向 `标准输出流（stdout）` 的 `可写流（Writable Stream）`（on fd `1`）。

例如，`console.log` 的定义：

    console.log = function(d) {
      process.stdout.write(d + '\n');
    };

`process.stderr` 和 `process.stdout` 和其他 Node 的流不同，因为它们不同被关闭（`end()` 会抛出异常），它们从不会产生 `finish` 事件并且写入的时候通常是阻塞的。

- 当它们引用常规文件或者TTY文件描述符时是阻塞的。

- 当它们引用管道：
  - 在 Linux/Unix 平台时，它们是阻塞的。
  - 在 Windows 平台时，它们同其他流一样是非阻塞的。

为了检测 Node 是否正在 TTY 的上下文空间中被运行着，可以从 `process.stderr`，`process.stdout` 或 `process.stdin` 上读取 `isTTY` 属性：

    $ node -p "Boolean(process.stdin.isTTY)"
    true
    $ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
    false

    $ node -p "Boolean(process.stdout.isTTY)"
    true
    $ node -p "Boolean(process.stdout.isTTY)" | cat
    false

更多信息参见 [TTY 文档](tty.html#tty_tty)。

## process.stderr

一个指向 `标准错误输出流（stderr）` 的 `可写流（Writable Stream）`（on fd `2`）。

`process.stderr` 和 `process.stdout` 和其他 Node 的流不同，因为它们不同被关闭（`end()` 会抛出异常），它们从不会产生 `finish` 事件并且写入的时候通常是阻塞的。

- 当它们引用常规文件或者TTY文件描述符时是阻塞的。

- 当它们引用管道：
  - 在 Linux/Unix 平台时，它们是阻塞的。
  - 在 Windows 平台时，它们同其他流一样是非阻塞的。

## process.stdin

一个指向 `标准输入流（stdin）` 的 `可读流（Readable Stream）`（on fd `0`）。

打开标准输入并监听两个事件的例子：

    process.stdin.setEncoding('utf8');

    process.stdin.on('readable', function() {
      var chunk = process.stdin.read();
      if (chunk !== null) {
        process.stdout.write('data: ' + chunk);
      }
    });

    process.stdin.on('end', function() {
      process.stdout.write('end');
    });

作为流（Stream）,`process.stdin` 也可以被用在“旧”模式中，因此兼容为 Node v0.10 之前的版本所写的脚本。

更多信息参见 [流兼容性](stream.html#stream_compatibility_with_older_node_versions) 。

在“旧”流模式中，stdin 流默认是被暂停的。所以必须调用 `process.stdin.resume()` 来读取它。注意，调用 `process.stdin.resume()` 之后流自己也会切换到“旧”模式。

如果你正开始一个新的项目，你应该优先考虑使用一个更“新”的流模式而不是“旧”的。

## process.argv

一个包含了命令行参数的数组。第一个元素肯定是 “node”，第二个是 JavaScript 文件的名字。之后便是其他附加之后的命令行参数。

    // print process.argv
    process.argv.forEach(function(val, index, array) {
      console.log(index + ': ' + val);
    });

运行结果：

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


## process.execPath

可以被运行的启动进程的绝对路径。

例如：

    /usr/local/bin/node


## process.execArgv

从可执行的启动进程的（命令）中获取的 Node 特定的命令行可选项的集合。这些可选项不会出现在 `process.argv` 中，并且不包括 Node 可执行的脚本文件的名称，或者其他跟在脚本名称之后的可选项。这些可选项有助于产生子进程，并且它们与父进程有相同的执行环境。

例如:

    $ node --harmony script.js --version

process.execArgv 的结果为：

    ['--harmony']

而 process.argv 的结果为：

    ['/usr/local/bin/node', 'script.js', '--version']


## process.abort()

这会使 Node 产生一个中止事件。这会引起 Node 退出并产生一个核心文件。

## process.chdir(directory)

改变进程的当前工作目录，如果失败抛出一个异常。

    console.log('Starting directory: ' + process.cwd());
    try {
      process.chdir('/tmp');
      console.log('New directory: ' + process.cwd());
    }
    catch (err) {
      console.log('chdir: ' + err);
    }

## process.cwd()

返回进程的当前工作目录。

    console.log('Current directory: ' + process.cwd());

## process.env

一个包含用户环境的对象，参见 environ(7)。

该对象看起来就像这样：

    { TERM: 'xterm-256color',
      SHELL: '/usr/local/bin/bash',
      USER: 'maciej',
      PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
      PWD: '/Users/maciej',
      EDITOR: 'vim',
      SHLVL: '1',
      HOME: '/Users/maciej',
      LOGNAME: 'maciej',
      _: '/usr/local/bin/node' }

你可以修改该对象，但是修改的部分并不会反应到进程志之外。这意味着以下这样使用是不会有效的：

    node -e 'process.env.foo = "bar"' && echo $foo

但这样会：

    process.env.foo = 'bar';
    console.log(process.env.foo);

## process.exit([code])

使用指定 `代码（code）`结束进程。如果代码被省略，则会使用表示“成功”的代码 `0`。

使用“失败”代码退出：

    process.exit(1);

执行 Node 的运行环境会看到该退出代码为 1。

## process.exitCode

A number which will be the process exit code, when the process either
exits gracefully, or is exited via `process.exit()` without specifying
a code.

Specifying a code to `process.exit(code)` will override any previous
setting of `process.exitCode`.


## process.getgid()

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Gets the group identity of the process. (See getgid(2).)
This is the numerical group id, not the group name.

    if (process.getgid) {
      console.log('Current gid: ' + process.getgid());
    }


## process.setgid(id)

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Sets the group identity of the process. (See setgid(2).)  This accepts either
a numerical ID or a groupname string. If a groupname is specified, this method
blocks while resolving it to a numerical ID.

    if (process.getgid && process.setgid) {
      console.log('Current gid: ' + process.getgid());
      try {
        process.setgid(501);
        console.log('New gid: ' + process.getgid());
      }
      catch (err) {
        console.log('Failed to set gid: ' + err);
      }
    }


## process.getuid()

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Gets the user identity of the process. (See getuid(2).)
This is the numerical userid, not the username.

    if (process.getuid) {
      console.log('Current uid: ' + process.getuid());
    }


## process.setuid(id)

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Sets the user identity of the process. (See setuid(2).)  This accepts either
a numerical ID or a username string.  If a username is specified, this method
blocks while resolving it to a numerical ID.

    if (process.getuid && process.setuid) {
      console.log('Current uid: ' + process.getuid());
      try {
        process.setuid(501);
        console.log('New uid: ' + process.getuid());
      }
      catch (err) {
        console.log('Failed to set uid: ' + err);
      }
    }


## process.getgroups()

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Returns an array with the supplementary group IDs. POSIX leaves it unspecified
if the effective group ID is included but node.js ensures it always is.


## process.setgroups(groups)

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Sets the supplementary group IDs. This is a privileged operation, meaning you
need to be root or have the CAP_SETGID capability.

The list can contain group IDs, group names or both.


## process.initgroups(user, extra_group)

Note: this function is only available on POSIX platforms (i.e. not Windows,
Android)

Reads /etc/group and initializes the group access list, using all groups of
which the user is a member. This is a privileged operation, meaning you need
to be root or have the CAP_SETGID capability.

`user` is a user name or user ID. `extra_group` is a group name or group ID.

Some care needs to be taken when dropping privileges. Example:

    console.log(process.getgroups());         // [ 0 ]
    process.initgroups('bnoordhuis', 1000);   // switch user
    console.log(process.getgroups());         // [ 27, 30, 46, 1000, 0 ]
    process.setgid(1000);                     // drop root gid
    console.log(process.getgroups());         // [ 27, 30, 46, 1000 ]


## process.version

A compiled-in property that exposes `NODE_VERSION`.

    console.log('Version: ' + process.version);

## process.versions

A property exposing version strings of node and its dependencies.

    console.log(process.versions);

Will print something like:

    { http_parser: '1.0',
      node: '0.10.4',
      v8: '3.14.5.8',
      ares: '1.9.0-DEV',
      uv: '0.10.3',
      zlib: '1.2.3',
      modules: '11',
      openssl: '1.0.1e' }

## process.config

An Object containing the JavaScript representation of the configure options
that were used to compile the current node executable. This is the same as
the "config.gypi" file that was produced when running the `./configure` script.

An example of the possible output looks like:

    { target_defaults:
       { cflags: [],
         default_configuration: 'Release',
         defines: [],
         include_dirs: [],
         libraries: [] },
      variables:
       { host_arch: 'x64',
         node_install_npm: 'true',
         node_prefix: '',
         node_shared_cares: 'false',
         node_shared_http_parser: 'false',
         node_shared_libuv: 'false',
         node_shared_v8: 'false',
         node_shared_zlib: 'false',
         node_use_dtrace: 'false',
         node_use_openssl: 'true',
         node_shared_openssl: 'false',
         strict_aliasing: 'true',
         target_arch: 'x64',
         v8_use_snapshot: 'true' } }

## process.kill(pid[, signal])

Send a signal to a process. `pid` is the process id and `signal` is the
string describing the signal to send.  Signal names are strings like
'SIGINT' or 'SIGHUP'.  If omitted, the signal will be 'SIGTERM'.
See [Signal Events](#process_signal_events) and kill(2) for more information.

Will throw an error if target does not exist, and as a special case, a signal of
`0` can be used to test for the existence of a process.

Note that just because the name of this function is `process.kill`, it is
really just a signal sender, like the `kill` system call.  The signal sent
may do something other than kill the target process.

Example of sending a signal to yourself:

    process.on('SIGHUP', function() {
      console.log('Got SIGHUP signal.');
    });

    setTimeout(function() {
      console.log('Exiting.');
      process.exit(0);
    }, 100);

    process.kill(process.pid, 'SIGHUP');

Note: When SIGUSR1 is received by Node.js it starts the debugger, see
[Signal Events](#process_signal_events).

## process.pid

The PID of the process.

    console.log('This process is pid ' + process.pid);


## process.title

Getter/setter to set what is displayed in 'ps'.

When used as a setter, the maximum length is platform-specific and probably
short.

On Linux and OS X, it's limited to the size of the binary name plus the
length of the command line arguments because it overwrites the argv memory.

v0.8 allowed for longer process title strings by also overwriting the environ
memory but that was potentially insecure/confusing in some (rather obscure)
cases.


## process.arch

What processor architecture you're running on: `'arm'`, `'ia32'`, or `'x64'`.

    console.log('This processor architecture is ' + process.arch);


## process.platform

What platform you're running on:
`'darwin'`, `'freebsd'`, `'linux'`, `'sunos'` or `'win32'`

    console.log('This platform is ' + process.platform);


## process.memoryUsage()

Returns an object describing the memory usage of the Node process
measured in bytes.

    var util = require('util');

    console.log(util.inspect(process.memoryUsage()));

This will generate:

    { rss: 4935680,
      heapTotal: 1826816,
      heapUsed: 650472 }

`heapTotal` and `heapUsed` refer to V8's memory usage.


## process.nextTick(callback)

* `callback` {Function}

Once the current event loop turn runs to completion, call the callback
function.

This is *not* a simple alias to `setTimeout(fn, 0)`, it's much more
efficient.  It runs before any additional I/O events (including
timers) fire in subsequent ticks of the event loop.

    console.log('start');
    process.nextTick(function() {
      console.log('nextTick callback');
    });
    console.log('scheduled');
    // Output:
    // start
    // scheduled
    // nextTick callback

This is important in developing APIs where you want to give the user the
chance to assign event handlers after an object has been constructed,
but before any I/O has occurred.

    function MyThing(options) {
      this.setupOptions(options);

      process.nextTick(function() {
        this.startDoingStuff();
      }.bind(this));
    }

    var thing = new MyThing();
    thing.getReadyForStuff();

    // thing.startDoingStuff() gets called now, not before.

It is very important for APIs to be either 100% synchronous or 100%
asynchronous.  Consider this example:

    // WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
    function maybeSync(arg, cb) {
      if (arg) {
        cb();
        return;
      }

      fs.stat('file', cb);
    }

This API is hazardous.  If you do this:

    maybeSync(true, function() {
      foo();
    });
    bar();

then it's not clear whether `foo()` or `bar()` will be called first.

This approach is much better:

    function definitelyAsync(arg, cb) {
      if (arg) {
        process.nextTick(cb);
        return;
      }

      fs.stat('file', cb);
    }

Note: the nextTick queue is completely drained on each pass of the
event loop **before** additional I/O is processed.  As a result,
recursively setting nextTick callbacks will block any I/O from
happening, just like a `while(true);` loop.

## process.umask([mask])

Sets or reads the process's file mode creation mask. Child processes inherit
the mask from the parent process. Returns the old mask if `mask` argument is
given, otherwise returns the current mask.

    var oldmask, newmask = 0022;

    oldmask = process.umask(newmask);
    console.log('Changed umask from: ' + oldmask.toString(8) +
                ' to ' + newmask.toString(8));


## process.uptime()

Number of seconds Node has been running.


## process.hrtime()

Returns the current high-resolution real time in a `[seconds, nanoseconds]`
tuple Array. It is relative to an arbitrary time in the past. It is not
related to the time of day and therefore not subject to clock drift. The
primary use is for measuring performance between intervals.

You may pass in the result of a previous call to `process.hrtime()` to get
a diff reading, useful for benchmarks and measuring intervals:

    var time = process.hrtime();
    // [ 1800216, 25 ]

    setTimeout(function() {
      var diff = process.hrtime(time);
      // [ 1, 552 ]

      console.log('benchmark took %d nanoseconds', diff[0] * 1e9 + diff[1]);
      // benchmark took 1000000527 nanoseconds
    }, 1000);


## process.mainModule

Alternate way to retrieve
[`require.main`](modules.html#modules_accessing_the_main_module).
The difference is that if the main module changes at runtime, `require.main`
might still refer to the original main module in modules that were required
before the change occurred. Generally it's safe to assume that the two refer
to the same module.

As with `require.main`, it will be `undefined` if there was no entry script.

[EventEmitter]: events.html#events_class_events_eventemitter
