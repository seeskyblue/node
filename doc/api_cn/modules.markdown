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

`module.exports` 对象是由模块系统创建的。但有时这难以接受的，许多人希望它们的模块是某个类的实例。为了做到这点，可将希望得到的模块输出对象赋值给 `module.exports`。注意，将希望得到的模块对象赋值给 `exports` 只是简单地重新定义了本地的 `exports` 变量，可能并不是你想要的那样。 

例如，假设我们在编写一个模块叫做 `a.js`：

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

然后，在一个别的模块文件中，我们可以这么做：

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });

注意，赋值给 `module.exports` 是必须立即执行完成的。不能放在任何回调函数中。如下这样是不会有效工作的：

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);

#### exports alias 输出（exports）别名

`exports` 变量在模块开始（执行）时，作为 `module.exports` 的引用可以获取到。与其他变量一样，如果你给他赋一个新的值之后，他再也不会绑定之前的值。

为了说明这个行为，想象以下是 `require()` 假想的实现：

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

原则上，如果 `exports` 和 `module.exports` 之间的关系对你来说太不确定，可以忽略 `exports` 而仅使用 `module.exports`。

### module.require(id)

* `id` {String}
* Return: {Object} 从已解析的模块返回的 `module.exports`

`module.require` 方法提供了一种加载模块的途径，犹如从源模块调用 `require()` 一般。

注意，为了这样操作，你必须先获得一个 `module` 对象的引用。由于 `require()` 返回的是 `module.exports`，并且 `module` 通常情况下 *只能* 在特定模块代码的内部获得。所以 `module` 对象必须被准确地输出，才能被使用。


### module.id

* {String}

模块的标识，通常是已完整解析后的文件路径（filename）。


### module.filename

* {String}

模块完整解析后的文件路径。


### module.loaded

* {Boolean}

是否模块已经完成加载，或者仍在加载过程中。


### module.parent

* {Module Object}

调用加载（require）本模块的模块对象。


### module.children

* {Array}

本模块调用加载（require）的模块对象。



## All Together... 综述...

<!-- type=misc -->

`require.resolve()` 函数可以获取调用 `require()` 时将会被加载的准确文件路径。

综上所述，以下是 `require.resolve()` 执行时高阶算法的伪代码：

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

## Loading from the global folders 从全局文件夹中加载

<!-- type=misc -->

If the `NODE_PATH` environment variable is set to a colon-delimited list
of absolute paths, then node will search those paths for modules if they
are not found elsewhere.  (Note: On Windows, `NODE_PATH` is delimited by
semicolons instead of colons.)
如果环境变量 `NODE_PATH` 被设置为冒号分割的绝对路径列表，那么 Node 如果在其他地方都找不到模块，就会搜索这些路径来加载模块。（注意，在Windows系统中，`NODE_PATH` 是被分号分割，而不是冒号分割。）

另外，Node 也会搜索如下位置：

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

其中，`$HOME` 是用户的 home 目录，`$PREFIX` 是由 Node 的配置的 `node_prefix`。

这些大多是由于历史的原因。强烈建议把依赖模块放在程序本地的 `node_modules` 文件夹中。这样可以被更快的加载，并且也更可靠。

## Accessing the main module 访问主模块

<!-- type=misc -->

当一个文件被 Node 直接执行，`require.main` 被设为它自己的 `module`。这意味着可以通过以下测试确定文件是否被直接运行：

    require.main === module

对于 `foo.js`文件，如果通过运行 `node foo.js` 将会返回 `true`，但通过运行 `require('./foo')` 会返回 `false`。

由于 `module` 提供 `filename` 属性（一般等于 `__filename`），当前程序的入口可以通过 `require.main.filename` 获取到。

## Addenda: Package Manager Tips 附录：包管理技巧

<!-- type=misc -->

Node的 `require()` 函数语法被设计得足够通用，以便支持各种健全的目录结构。包管理器如 `dpkg`、`rpm` 和 `npm`等但愿可以无需修改 Node 模块就能创建本地包。

以下，我们给出一个目录结构建议可以满足以上要求：

让我们假设要让 `/usr/lib/node/<some-package>/<some-version>` 文件夹包含一个特定版本包的内容。 

包可以依赖另一个包。为了按照 `foo` 包，你可能需要按照一个特定版本的 `bar` 包。但 `bar` 包可能自己也有依赖，并且在某些情况下，这些依赖可以相互不兼容，甚至是循环依赖。

由于 Node 查找任何模块的 `realpath`（绝对路径），然后按之前描述在 `node_modules` 文件夹中查找他们的依赖，这种情况按照如下结构会非常容易去处理：

* `/usr/lib/node/foo/1.2.3/` - `foo` 包的内容, 版本为 1.2.3。
* `/usr/lib/node/bar/4.3.2/` - `bar` 包的内容，被 `foo` 包依赖。
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - 到 `/usr/lib/node/bar/4.3.2/` 的符号链接。
* `/usr/lib/node/bar/4.3.2/node_modules/*` - `bar` 依赖包的符号链接。

因此，即使遇到循环依赖，或者有依赖冲突，每个模块都可以获得它可以使用版本的依赖。

当 `foo` 包中的代码执行 `foo`，会得到符号链接到 `/usr/lib/node/foo/1.2.3/node_modules/bar` 的版本。而当 `bar` 包中的代码调用 `require('quux')`，会得到符号链接到 `/usr/lib/node/bar/4.3.2/node_modules/quux` 的版本。

此外，为使模块查找过程更加优化，相比于把包直接放在 `/usr/lib/node` 目录，我们可以把他们放到 `/usr/lib/node_modules/<name>/<version>`。这样可以不用麻烦 Node 再去 `/usr/node_modules` 或 `/node_modules` 查找未找到的依赖。

为了使模块能够在 Node 的 REPL（即时结果输出）环境中被访问，将 `/usr/lib/node_modules` 文件夹添加到 `$NODE_PATH` 环境变量中会很有用。由于模块使用 `node_modules` 文件夹查找都是基于调用 `require()` 文件绝对路径的相对路径，而包文件本身可能再任何地方。
