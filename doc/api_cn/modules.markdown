# Modules 模块

    稳定度: 5 - 锁定

<!--name=module-->

Node有一个简单的模块加载系统。在Node中，文件和模块是一一对应的。例如，`foo.js` 加载了同目录下的 `circle.js` 模块。 

`foo.js` 代码为:

    var circle = require('./circle.js');
    console.log( 'The area of a circle of radius 4 is '
               + circle.area(4));

`circle.js` 代码为:

    var PI = Math.PI;

    exports.area = function (r) {
      return PI * r * r;
    };

    exports.circumference = function (r) {
      return 2 * PI * r;
    };

`circle.js` 模块输出的是  `area()` 和 `circumference()` 函数。如需要添加函数或者对象到根模块，可以将他们添加到特殊对象 `exports`。

模块本地的变量对于模块来说是私有的（private），就像这个模块是被function关键字包装过。在这个例子中，变量 `PI` 对于 `circle.js` 模块来说是私有的。

如果你需要把模块输出的根作为一个函数（例如构造函数），或者你希望一次输出一个完整的对象而不是每个属性都构造一次，那就使用 `module.exports` 来代替 `exports`。

如下，`bar.js` 使用了以构造函数作为输出的 `square` 模块：

    var square = require('./square.js');
    var mySquare = square(2);
    console.log('The area of my square is ' + mySquare.area());

`square` 模块在 `square.js` 文件中定义:

    // assigning to exports will not modify module, must use module.exports
    module.exports = function(width) {
      return {
        area: function() {
          return width * width;
        }
      };
    }

模块系统是通过 `require("module")` 的模式来实现的。

## Cycles 循环依赖

<!--type=misc-->

当有循环 `require()` 调用时，模块返回的时候可能还未执行结束。

考虑一下这样的情况：

`a.js`:

    console.log('a starting');
    exports.done = false;
    var b = require('./b.js');
    console.log('in a, b.done = %j', b.done);
    exports.done = true;
    console.log('a done');

`b.js`:

    console.log('b starting');
    exports.done = false;
    var a = require('./a.js');
    console.log('in b, a.done = %j', a.done);
    exports.done = true;
    console.log('b done');

`main.js`:

    console.log('main starting');
    var a = require('./a.js');
    var b = require('./b.js');
    console.log('in main, a.done=%j, b.done=%j', a.done, b.done);

当 `main.js` 开始加载 `a.js` 后，接着 `a.js` 开始加载 `b.js`。在这个时刻，`b.js` 开始试图加载 `a.js`。为了避免无限循环的加载，一个 `a.js` **未执行完成复制品** 的输出对象被返回给 `b.js` 模块。然后 `b.js` 模块完成了加载，然后它的 `exports` 对象被返回给 `a.js` 模块。

此时，`main.js` 已经完成了加载这两个模块，并且他们都已经执行完成。程序的输出结果为：

    $ node main.js
    main starting
    a starting
    b starting
    in b, a.done = false
    b done
    in a, b.done = true
    a done
    in main, a.done=true, b.done=true

如果你的程序中有模块循环依赖，确保有相应的（循环依赖加载顺序）准备。

## Core Modules 核心模块

<!--type=misc-->

Node有几个模块是被编译为二进制的（核心模块）。这些模块在本文档的其他地方有更加详细的描述。

这些核心模块被定义在Node源代码中的 `lib/` 文件夹内。

当核心模块的标识被传递给 `require()` 时，核心模块总是优先被加载。例如，`require('http')`总是返回内置的HTTP核心模块，即使存在一个同样名为 `http` 的文件（模块）。

## File Modules 文件模块

<!--type=misc-->

如果按文件名精确查找未找到文件，Node会试图在文件名之后添加 `.js` 后缀来加载。如还未找到，则添加 `.json` 加载，最后还会尝试添加 `.node` 后缀加载。

`.js` 文件按照 Javascript 文件解释，`.json` 文件被解析为 JSON 文本文件。`.node.` 文件被解释为已由 `dlopen` 加载了的编译过的插件模块。

如果一个模块（名）由 `'/'` 前缀开头，表示这是一个（模块）文件的绝对路径。例如，`require('/home/marco/foo.js')` 将会加载在 `/home/marco/` 目录下的 `foo.js` 文件。

如果一个模块（名）由 `'./'` 前缀开头，表示这是一个（模块）文件的相对路径，相对于调用 `require()` 的模块文件所在路径。也就是说，如果在 `foo.js` 中调用 `require('./circle')` 时，`circle.js` 必须与它在同一个目录中才能找到。

如果（模块）文件描述中没有 `'/'` 或 `'./'` 开头，该模块要不就是一个“核心模块”或者是从 `node_modules` 文件夹中被加载。

如果传入的文件路径不存在，`require()` 会抛出一个Error异常，并且该异常的 `code` 属性被设为 `'MODULE_NOT_FOUND'`。

## Loading from `node_modules` Folders 从 `node_modules` 文件夹加载

<!--type=misc-->

如果一个传入 `require()` 的模块标识不是一个本地的模块，也不是以 `'/'`、 `'../'`，或 `'./'` 开头。那么Node会在当前模块的父目录（路径标识），加上 `/node_modules`（路径标识），然后尝试从该位置加载模块。

如果模块未找到，那么Node继续尝试再上一层目录中的 `/node_modules` 文件夹中寻找模块加载，直至文件系统的根目录。

例如，如果模块文件为 `'/home/ry/projects/foo.js'` 调用 `require('bar.js')`，那么Node会按照这样的顺序在如下位置进行查找：

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

这样允许程序将它们依赖集中起来，以免发生冲突。

可以通过在模块名之后添加路径标识后缀来加载（require）特定的文件或分布式的子模块。例如 `require('example-module/path/to/file')` 会被解析为 `path/to/file` 路径对应 `example-module` 模块所在的位置。该后缀路径标识同样遵循之前模块解析加载的语法。

## Folders as Modules 模块文件夹

<!--type=misc-->

可以很方便地将程序和库放入独立的文件夹中，并提供单一的入口来指向它。有三种方法可以使用一个文件夹（路径标识）作为传入 `require()` 的参数。

第一个方法是在该文件夹的根目录中创建一个 `package.json` 文件，它指定了一个 `main` 模块。例如，一个 `package.json` 可能看起来像这样：

    { "name" : "some-library",
      "main" : "./lib/some-library.js" }

如果该文件在目录 `./some-library` 中，那么 `require('./some-library')` 会试图加载 `./some-library/lib/some-library.js`。

有关 Node 加载 `package.json` 文件，你还需要知道以下这些。

如果在目录中没有发现 `package.json` 文件，那么 Node 会试图从该目录中加载 `index.js` 或 `index.node` 文件。例如，如果在上面这个例子中没有 `package.json` 文件，那么 `require('./some-library')` 会试图加载：

* `./some-library/index.js`
* `./some-library/index.node`

## Caching 缓存

<!--type=misc-->

模块在第一次被加载之后被缓存。这意味着（除此以外）每一次调用 `require('foo')` 其实会返回同一个对象，如果每次都是解析同一个文件的话。

多次调用 `require('foo')` 不会让模块代码被执行多次。这是一个重要的特性。利用它可以使“部分完成”的对象被返回，因此才允许传递依赖被加载，即使它们会导致循环依赖。

如果你需要让一个模块执行多次代码，那么输出一个函数，然后调用该函数来实现。

### Module Caching Caveats 模块缓存警告

<!--type=misc-->

模块是基于解析它们的文件路径标识来缓存的。由于模块因调用它们的模块所在位置不同，可能被解析为不同的文件路径标识（从 `node_modules` 文件夹中被加载）。因此如果解析到不同文件是，就不能*保证* `require('foo')` 总是会返回绝对相同的对象。

## The `module` Object `'module'` 对象

<!-- type=var -->
<!-- name=module -->

* {Object}

在每个模块内，免定义变量 `module` 是一个对代表当前模块对象的引用。为了方便，`module.exports` 也可以通过模块全局变量 `exports` 访问到。`module` 实际上并不是一个全局变量，而是每个模块的本地变量。

### module.exports

* {Object}

The `module.exports` object is created by the Module system. Sometimes this is not
acceptable; many want their module to be an instance of some class. To do this,
assign the desired export object to `module.exports`. Note that assigning the
desired object to `exports` will simply rebind the local `exports` variable,
which is probably not what you want to do.

`module.exports` 对象是由模块系统创建的。但有时这难以接受的，许多人希望它们的模块是某个类的实例。为了做到这一点，将希望

For example suppose we were making a module called `a.js`

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

Then in another file we could do

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });


Note that assignment to `module.exports` must be done immediately. It cannot be
done in any callbacks.  This does not work:

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);

#### exports alias

The `exports` variable that is available within a module starts as a reference
to `module.exports`. As with any variable, if you assign a new value to it, it
is no longer bound to the previous value.

To illustrate the behaviour, imagine this hypothetical implementation of
`require()`:

    function require(...) {
      // ...
      function (module, exports) {
        // Your module code here
        exports = some_func;        // re-assigns exports, exports is no longer
                                    // a shortcut, and nothing is exported.
        module.exports = some_func; // makes your module export 0
      } (module, module.exports);
      return module;
    }

As a guideline, if the relationship between `exports` and `module.exports`
seems like magic to you, ignore `exports` and only use `module.exports`.

### module.require(id)

* `id` {String}
* Return: {Object} `module.exports` from the resolved module

The `module.require` method provides a way to load a module as if
`require()` was called from the original module.

Note that in order to do this, you must get a reference to the `module`
object.  Since `require()` returns the `module.exports`, and the `module` is
typically *only* available within a specific module's code, it must be
explicitly exported in order to be used.


### module.id

* {String}

The identifier for the module.  Typically this is the fully resolved
filename.


### module.filename

* {String}

The fully resolved filename to the module.


### module.loaded

* {Boolean}

Whether or not the module is done loading, or is in the process of
loading.


### module.parent

* {Module Object}

The module that required this one.


### module.children

* {Array}

The module objects required by this one.



## All Together...

<!-- type=misc -->

To get the exact filename that will be loaded when `require()` is called, use
the `require.resolve()` function.

Putting together all of the above, here is the high-level algorithm
in pseudocode of what require.resolve does:

    require(X) from module at path Y
    1. If X is a core module,
       a. return the core module
       b. STOP
    2. If X begins with './' or '/' or '../'
       a. LOAD_AS_FILE(Y + X)
       b. LOAD_AS_DIRECTORY(Y + X)
    3. LOAD_NODE_MODULES(X, dirname(Y))
    4. THROW "not found"

    LOAD_AS_FILE(X)
    1. If X is a file, load X as JavaScript text.  STOP
    2. If X.js is a file, load X.js as JavaScript text.  STOP
    3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
    4. If X.node is a file, load X.node as binary addon.  STOP

    LOAD_AS_DIRECTORY(X)
    1. If X/package.json is a file,
       a. Parse X/package.json, and look for "main" field.
       b. let M = X + (json main field)
       c. LOAD_AS_FILE(M)
    2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
    3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
    4. If X/index.node is a file, load X/index.node as binary addon.  STOP

    LOAD_NODE_MODULES(X, START)
    1. let DIRS=NODE_MODULES_PATHS(START)
    2. for each DIR in DIRS:
       a. LOAD_AS_FILE(DIR/X)
       b. LOAD_AS_DIRECTORY(DIR/X)

    NODE_MODULES_PATHS(START)
    1. let PARTS = path split(START)
    2. let I = count of PARTS - 1
    3. let DIRS = []
    4. while I >= 0,
       a. if PARTS[I] = "node_modules" CONTINUE
       c. DIR = path join(PARTS[0 .. I] + "node_modules")
       b. DIRS = DIRS + DIR
       c. let I = I - 1
    5. return DIRS

## Loading from the global folders

<!-- type=misc -->

If the `NODE_PATH` environment variable is set to a colon-delimited list
of absolute paths, then node will search those paths for modules if they
are not found elsewhere.  (Note: On Windows, `NODE_PATH` is delimited by
semicolons instead of colons.)

Additionally, node will search in the following locations:

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

Where `$HOME` is the user's home directory, and `$PREFIX` is node's
configured `node_prefix`.

These are mostly for historic reasons.  You are highly encouraged to
place your dependencies locally in `node_modules` folders.  They will be
loaded faster, and more reliably.

## Accessing the main module

<!-- type=misc -->

When a file is run directly from Node, `require.main` is set to its
`module`. That means that you can determine whether a file has been run
directly by testing

    require.main === module

For a file `foo.js`, this will be `true` if run via `node foo.js`, but
`false` if run by `require('./foo')`.

Because `module` provides a `filename` property (normally equivalent to
`__filename`), the entry point of the current application can be obtained
by checking `require.main.filename`.

## Addenda: Package Manager Tips

<!-- type=misc -->

The semantics of Node's `require()` function were designed to be general
enough to support a number of sane directory structures. Package manager
programs such as `dpkg`, `rpm`, and `npm` will hopefully find it possible to
build native packages from Node modules without modification.

Below we give a suggested directory structure that could work:

Let's say that we wanted to have the folder at
`/usr/lib/node/<some-package>/<some-version>` hold the contents of a
specific version of a package.

Packages can depend on one another. In order to install package `foo`, you
may have to install a specific version of package `bar`.  The `bar` package
may itself have dependencies, and in some cases, these dependencies may even
collide or form cycles.

Since Node looks up the `realpath` of any modules it loads (that is,
resolves symlinks), and then looks for their dependencies in the
`node_modules` folders as described above, this situation is very simple to
resolve with the following architecture:

* `/usr/lib/node/foo/1.2.3/` - Contents of the `foo` package, version 1.2.3.
* `/usr/lib/node/bar/4.3.2/` - Contents of the `bar` package that `foo`
  depends on.
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - Symbolic link to
  `/usr/lib/node/bar/4.3.2/`.
* `/usr/lib/node/bar/4.3.2/node_modules/*` - Symbolic links to the packages
  that `bar` depends on.

Thus, even if a cycle is encountered, or if there are dependency
conflicts, every module will be able to get a version of its dependency
that it can use.

When the code in the `foo` package does `require('bar')`, it will get the
version that is symlinked into `/usr/lib/node/foo/1.2.3/node_modules/bar`.
Then, when the code in the `bar` package calls `require('quux')`, it'll get
the version that is symlinked into
`/usr/lib/node/bar/4.3.2/node_modules/quux`.

Furthermore, to make the module lookup process even more optimal, rather
than putting packages directly in `/usr/lib/node`, we could put them in
`/usr/lib/node_modules/<name>/<version>`.  Then node will not bother
looking for missing dependencies in `/usr/node_modules` or `/node_modules`.

In order to make modules available to the node REPL, it might be useful to
also add the `/usr/lib/node_modules` folder to the `$NODE_PATH` environment
variable.  Since the module lookups using `node_modules` folders are all
relative, and based on the real path of the files making the calls to
`require()`, the packages themselves can be anywhere.
