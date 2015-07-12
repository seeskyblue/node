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

- `SIGTERM` 和 `SIGINT` 在非 Windows 平台上有默认处理，即在退出状态代码为 `128 + 信号代码` 时重置终端模式。当这些信号有创建监听是，它们的默认行为将被取消。
  （Node 将不会退出）。
  
- `SIGPIPE` is ignored by default, it can have a listener installed.
- `SIGHUP` is generated on Windows when the console window is closed, and on other
  platforms under various similar conditions, see signal(7). It can have a
  listener installed, however node will be unconditionally terminated by Windows
  about 10 seconds later. On non-Windows platforms, the default behaviour of
  `SIGHUP` is to terminate node, but once a listener has been installed its
  default behaviour will be removed.
- `SIGTERM` is not supported on Windows, it can be listened on.
- `SIGINT` from the terminal is supported on all platforms, and can usually be
  generated with `CTRL+C` (though this may be configurable). It is not generated
  when terminal raw mode is enabled.
- `SIGBREAK` is delivered on Windows when `CTRL+BREAK` is pressed, on non-Windows
  platforms it can be listened on, but there is no way to send or generate it.
- `SIGWINCH` is delivered when the console has been resized. On Windows, this will
  only happen on write to the console when the cursor is being moved, or when a
  readable tty is used in raw mode.
- `SIGKILL` cannot have a listener installed, it will unconditionally terminate
  node on all platforms.
- `SIGSTOP` cannot have a listener installed.

Note that Windows does not support sending Signals, but node offers some
emulation with `process.kill()`, and `child_process.kill()`:
- Sending signal `0` can be used to search for the existence of a process
- Sending `SIGINT`, `SIGTERM`, and `SIGKILL` cause the unconditional exit of the
  target process.

## process.stdout

A `Writable Stream` to `stdout` (on fd `1`).

Example: the definition of `console.log`

    console.log = function(d) {
      process.stdout.write(d + '\n');
    };

`process.stderr` and `process.stdout` are unlike other streams in Node in
that they cannot be closed (`end()` will throw), they never emit the `finish`
event and that writes are usually blocking.

- They are blocking in the case that they refer to regular files or TTY file
  descriptors.
- In the case they refer to pipes:
  - They are blocking in Linux/Unix.
  - They are non-blocking like other streams in Windows.

To check if Node is being run in a TTY context, read the `isTTY` property
on `process.stderr`, `process.stdout`, or `process.stdin`:

    $ node -p "Boolean(process.stdin.isTTY)"
    true
    $ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
    false

    $ node -p "Boolean(process.stdout.isTTY)"
    true
    $ node -p "Boolean(process.stdout.isTTY)" | cat
    false

See [the tty docs](tty.html#tty_tty) for more information.

## process.stderr

A writable stream to stderr (on fd `2`).

`process.stderr` and `process.stdout` are unlike other streams in Node in
that they cannot be closed (`end()` will throw), they never emit the `finish`
event and that writes are usually blocking.

- They are blocking in the case that they refer to regular files or TTY file
  descriptors.
- In the case they refer to pipes:
  - They are blocking in Linux/Unix.
  - They are non-blocking like other streams in Windows.


## process.stdin

A `Readable Stream` for stdin (on fd `0`).

Example of opening standard input and listening for both events:

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

As a Stream, `process.stdin` can also be used in "old" mode that is compatible
with scripts written for node prior v0.10.
For more information see
[Stream compatibility](stream.html#stream_compatibility_with_older_node_versions).

In "old" Streams mode the stdin stream is paused by default, so one
must call `process.stdin.resume()` to read from it. Note also that calling
`process.stdin.resume()` itself would switch stream to "old" mode.

If you are starting a new project you should prefer a more recent "new" Streams
mode over "old" one.

## process.argv

An array containing the command line arguments.  The first element will be
'node', the second element will be the name of the JavaScript file.  The
next elements will be any additional command line arguments.

    // print process.argv
    process.argv.forEach(function(val, index, array) {
      console.log(index + ': ' + val);
    });

This will generate:

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


## process.execPath

This is the absolute pathname of the executable that started the process.

Example:

    /usr/local/bin/node


## process.execArgv

This is the set of node-specific command line options from the
executable that started the process.  These options do not show up in
`process.argv`, and do not include the node executable, the name of
the script, or any options following the script name. These options
are useful in order to spawn child processes with the same execution
environment as the parent.

Example:

    $ node --harmony script.js --version

results in process.execArgv:

    ['--harmony']

and process.argv:

    ['/usr/local/bin/node', 'script.js', '--version']


## process.abort()

This causes node to emit an abort. This will cause node to exit and
generate a core file.

## process.chdir(directory)

Changes the current working directory of the process or throws an exception if that fails.

    console.log('Starting directory: ' + process.cwd());
    try {
      process.chdir('/tmp');
      console.log('New directory: ' + process.cwd());
    }
    catch (err) {
      console.log('chdir: ' + err);
    }



## process.cwd()

Returns the current working directory of the process.

    console.log('Current directory: ' + process.cwd());


## process.env

An object containing the user environment. See environ(7).

An example of this object looks like:

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

You can write to this object, but changes won't be reflected outside of your
process. That means that the following won't work:

    node -e 'process.env.foo = "bar"' && echo $foo

But this will:

    process.env.foo = 'bar';
    console.log(process.env.foo);


## process.exit([code])

Ends the process with the specified `code`.  If omitted, exit uses the
'success' code `0`.

To exit with a 'failure' code:

    process.exit(1);

The shell that executed node should see the exit code as 1.


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
