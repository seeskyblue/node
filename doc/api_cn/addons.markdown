# Addons 插件

插件是动态链接并共享的对象。他们可以提供 C 和 C++ 库的访问支持。该API（现在）相当复杂，包括多个库的知识：

 - V8 JavaScript, C++库。用来作为处理JavaScript的接口，包括：创建对象，调用函数等。文档大多数在 `v8.h` 头文件中（在 Node 源代码中的 `deps/v8/include/v8.h`），也可以 [在线](http://izs.me/v8-docs/main.html) 获取。

 - [libuv](https://github.com/joyent/libuv), C语言事件循环库。任何时候，当需要等待定时器或者等待收到一个信号，让一个文件描述符变为可读的时候就需要 libuv 接口。也就是说，如果你打算执行任何 I/O 操作的时候，libuv 就需要被用到。

 - 内部 Node 库。最重要的是 `node::ObjectWrap` 类，你很可能会想要获得。

 - 其他的，查看 `deps/` 来获取其他有用信息。

Node 静态编译所有他的依赖为可执行的（库）。当编译你的模块时，不需要担心任何对这些库的引用。

所有接下来的例子可以通过点击 [这里](https://github.com/rvagg/node-addon-examples) 下载，也许可以作为（创建）你自己插件的一个起点。

## Hello world

开始，让我们制作一个 C++ 小插件，等价于以下 JavaScript 代码：

    module.exports.hello = function() { return 'world'; };

首先，我们创建一个 `hello.cc` 文件：

    // hello.cc
    #include <node.h>

    using namespace v8;

    void Method(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));
    }

    void init(Handle<Object> exports) {
      NODE_SET_METHOD(exports, "hello", Method);
    }

    NODE_MODULE(addon, init)

注意，所有的 Node 插件必须输出一个初始化（initialization）函数：

    void Initialize (Handle<Object> exports);
    NODE_MODULE(module_name, Initialize)

在 `NODE_MODULE` 之后没有分号，因为他不是一个函数（参见 `node.h`）。

`module_name` 需要符合最终便以为二进制文件的文件名（去除了 `.node` 后缀）。 

源代码需要被做成 `addon.node`，二进制的插件。为了这么做，我们需要创建一个 `binding.gyp` 文件，他使用类似JSON的格式来描述创建你的模块的配置信息。该文件通过 [node-gyp](https://github.com/TooTallNate/node-gyp) 来编译。

    {
      "targets": [
        {
          "target_name": "addon",
          "sources": [ "hello.cc" ]
        }
      ]
    }

下一步就是生成合适的项目来为当前平台创建文件。可以使用 `node-gyp configure`。

现在，你可以在 `build/` 目录中使用 `Makefile`（在Unix平台）或者 `vcxproj`（在Windows平台）来创建文件。之后调用 `node-gyp build` 命令。

现在，你有了编译过的 `.node` 绑定文件了！最终编译过的绑定文件会在 `build/Release/` 目录中。

你现在可以在 Node 项目中的 `hello.js` 文件中，通过告诉 `require` 最新创建的 `hello.node` 模块，来使用二进制的插件：

    // hello.js
    var addon = require('./build/Release/addon');

    console.log(addon.hello()); // 'world'

在接下来的文档中提供更多有关插件（设计）模式参考，也可以访问 <https://github.com/arturadib/node-qt> 获取更多产品案例。


## Addon patterns 插件模式

以下是一些插件（设计）模式，以帮助你起步。可以查询在线 [V8 参考](http://izs.me/v8-docs/main.html) 获取有关各种 V8 的调用。也可以参考 V8 的 [内置手册](http://code.google.com/apis/v8/embed.html) 的说明来了解它所使用的一些概念，例如句柄（handles）、范围（scopes）、函数模板（function templates）等。

想要运行这些示例，你需要使用 `node-gyp` 编译它们。创建如下的 `binding.gyp` 文件：

    {
      "targets": [
        {
          "target_name": "addon",
          "sources": [ "addon.cc" ]
        }
      ]
    }

当有多个 `.cc` 文件的时候，只需要简单的把文件名添加到 `sources` 数组中，例如：

    "sources": ["addon.cc", "myexample.cc"]

现在，你已经准备好 `binding.gyp` 文件了，可以设置和创建该插件：

    $ node-gyp configure build


### Function arguments 函数参数

接下来的（设计）模式展现了如何从 JavaScript 的函数调用中读取参数并返回一个结果。这是主要的并且也是唯一需要的 `addon.cc` 代码：

    // addon.cc
    #include <node.h>

    using namespace v8;

    void Add(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.Length() < 2) {
        isolate->ThrowException(Exception::TypeError(
            String::NewFromUtf8(isolate, "Wrong number of arguments")));
        return;
      }

      if (!args[0]->IsNumber() || !args[1]->IsNumber()) {
        isolate->ThrowException(Exception::TypeError(
            String::NewFromUtf8(isolate, "Wrong arguments")));
        return;
      }

      double value = args[0]->NumberValue() + args[1]->NumberValue();
      Local<Number> num = Number::New(isolate, value);

      args.GetReturnValue().Set(num);
    }

    void Init(Handle<Object> exports) {
      NODE_SET_METHOD(exports, "add", Add);
    }

    NODE_MODULE(addon, Init)

可以执行以下 JavaScript 片段测试：

    // test.js
    var addon = require('./build/Release/addon');

    console.log( 'This should be eight:', addon.add(3,5) );


### Callbacks 回调

你可以将 JavaScript 函数传递给 C++ 函数并且执行它，以下是 `addon.cc` 代码：

    // addon.cc
    #include <node.h>

    using namespace v8;

    void RunCallback(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<Function> cb = Local<Function>::Cast(args[0]);
      const unsigned argc = 1;
      Local<Value> argv[argc] = { String::NewFromUtf8(isolate, "hello world") };
      cb->Call(isolate->GetCurrentContext()->Global(), argc, argv);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", RunCallback);
    }

    NODE_MODULE(addon, Init)

注意，在这个例子中的 `Init()` 函数有两个参数，第二个参数接收完整的 `module` 对象。这就允许插件使用单个函数完全覆盖 `exports`，而不是将函数作为 `exports` 的属性。

可以执行以下 JavaScript 片段测试：

    // test.js
    var addon = require('./build/Release/addon');

    addon(function(msg){
      console.log(msg); // 'hello world'
    });


### Object factory 对象工厂

可以参考 `addon.cc` 的（设计）模式，在 C++ 函数中创建并返回新的对象。`addon.cc` 中返回一个拥有 `msg` 属性的对象，反映了传递给 `createObject()` 的字符串：

    // addon.cc
    #include <node.h>

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<Object> obj = Object::New(isolate);
      obj->Set(String::NewFromUtf8(isolate, "msg"), args[0]->ToString());

      args.GetReturnValue().Set(obj);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", CreateObject);
    }

    NODE_MODULE(addon, Init)

通过以下 JavaScript 代码测试：

    // test.js
    var addon = require('./build/Release/addon');

    var obj1 = addon('hello');
    var obj2 = addon('world');
    console.log(obj1.msg+' '+obj2.msg); // 'hello world'


### Function factory 函数工厂

该（设计）模式展示了如何在 C++ 函数中创建并返回一个 JavaScript 函数：

    // addon.cc
    #include <node.h>

    using namespace v8;

    void MyFunction(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      args.GetReturnValue().Set(String::NewFromUtf8(isolate, "hello world"));
    }

    void CreateFunction(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, MyFunction);
      Local<Function> fn = tpl->GetFunction();

      // omit this to make it anonymous
      fn->SetName(String::NewFromUtf8(isolate, "theFunction"));

      args.GetReturnValue().Set(fn);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", CreateFunction);
    }

    NODE_MODULE(addon, Init)

测试：

    // test.js
    var addon = require('./build/Release/addon');

    var fn = addon();
    console.log(fn()); // 'hello world'


### Wrapping C++ objects 封装 C++ 对象

这里我们将创建一个 C++ 对象/类 `MyObject` 的封装，可以在 JavaScript中通过 `new` 操作实例化。

首先创建主模块 `addon.cc`：

    // addon.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    void InitAll(Handle<Object> exports) {
      MyObject::Init(exports);
    }

    NODE_MODULE(addon, InitAll)

然后在 `myobject.h` 中，使封装对象/类继承于 `node::ObjectWrap`：

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init(v8::Handle<v8::Object> exports);

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

然后在 `myobject.cc` 中实现你需要暴露的变量方法。这里，我们暴露 `plusOne` 方法，通过将它添加到构造器的原型：

    // myobject.cc
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init(Handle<Object> exports) {
      Isolate* isolate = Isolate::GetCurrent();

      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      // Prototype
      NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

      constructor.Reset(isolate, tpl->GetFunction());
      exports->Set(String::NewFromUtf8(isolate, "MyObject"),
                   tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
      obj->value_ += 1;

      args.GetReturnValue().Set(Number::New(isolate, obj->value_));
    }

测试：

    // test.js
    var addon = require('./build/Release/addon');

    var obj = new addon.MyObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13

### Factory of wrapped objects 封装对象工厂

当你想要创建本地对象，但不想在 JavaScript 中显式地通过 `new` 操作来实例化它们，那么这就会变得有用。比如：

    var obj = addon.createObject();
    // instead of:
    // var obj = new addon.Object();

让我们在 `addon.cc` 中添加 `createObject` 方法：

    // addon.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      MyObject::NewInstance(args);
    }

    void InitAll(Handle<Object> exports, Handle<Object> module) {
      MyObject::Init();

      NODE_SET_METHOD(module, "exports", CreateObject);
    }

    NODE_MODULE(addon, InitAll)

在 `myobject.h` 文件中，我们让静态方法 `NewInstance` 来实例化对象（即，相当于 JavaScript 中 `new` 的工作）：

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

实现有点类似上面，在 `myobject.cc` 中：

    // myobject.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init() {
      Isolate* isolate = Isolate::GetCurrent();
      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      // Prototype
      NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

      constructor.Reset(isolate, tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> instance = cons->NewInstance(argc, argv);

      args.GetReturnValue().Set(instance);
    }

    void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
      obj->value_ += 1;

      args.GetReturnValue().Set(Number::New(isolate, obj->value_));
    }

测试:

    // test.js
    var createObject = require('./build/Release/addon');

    var obj = createObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13

    var obj2 = createObject(20);
    console.log( obj2.plusOne() ); // 21
    console.log( obj2.plusOne() ); // 22
    console.log( obj2.plusOne() ); // 23


### Passing wrapped objects around

In addition to wrapping and returning C++ objects, you can pass them around
by unwrapping them with Node's `node::ObjectWrap::Unwrap` helper function.
In the following `addon.cc` we introduce a function `add()` that can take on two
`MyObject` objects:

除了封装和返回 C++ 对象，也可以通过

    // addon.cc
    #include <node.h>
    #include <node_object_wrap.h>
    #include "myobject.h"

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      MyObject::NewInstance(args);
    }

    void Add(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
          args[0]->ToObject());
      MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
          args[1]->ToObject());

      double sum = obj1->value() + obj2->value();
      args.GetReturnValue().Set(Number::New(isolate, sum));
    }

    void InitAll(Handle<Object> exports) {
      MyObject::Init();

      NODE_SET_METHOD(exports, "createObject", CreateObject);
      NODE_SET_METHOD(exports, "add", Add);
    }

    NODE_MODULE(addon, InitAll)

To make things interesting we introduce a public method in `myobject.h` so we
can probe private values after unwrapping the object:

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
      inline double value() const { return value_; }

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

The implementation of `myobject.cc` is similar as before:

    // myobject.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init() {
      Isolate* isolate = Isolate::GetCurrent();

      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      constructor.Reset(isolate, tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> instance = cons->NewInstance(argc, argv);

      args.GetReturnValue().Set(instance);
    }

Test it with:

    // test.js
    var addon = require('./build/Release/addon');

    var obj1 = addon.createObject(10);
    var obj2 = addon.createObject(20);
    var result = addon.add(obj1, obj2);

    console.log(result); // 30
