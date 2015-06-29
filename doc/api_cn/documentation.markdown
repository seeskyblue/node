# 关于本文

<!-- type=misc -->

本文的目的是从参考和概念的角度全面解释Node.js的API，每个章节描述一个内置模块或高级概念。

属性类型、方法参数和事件处理的参数会被详细列在对应的标题下。

每个`.html`文件都有一个对应的内容相同的结构化的`.json`文件来表示。这个特性还是实验性质的，以便于集成开发工具（IDE）或其他工具希望能对本文档做操作。

每个`.html`文件和`.json`文件是基于对应的在node源代码中`doc/api/`文件夹中的`.markdown`文件生成的。使用`tools/doc/generate.js` 程序进行生成。HTML模板文件是`doc/template.html`。

## 稳定度

<!--type=misc-->

贯穿全文，你会看到章节稳定度的标记。Node.js的API在某种程度上是变化中的，当他成长的时候，某些部分会相对其他部分更可靠。有些被证明过非常可靠，几乎不可能再被修改。也有些是新增的实验性的，或是已知有危险的并在重新设计的过程中。

稳定度描述如下：

```
稳定度：0 - 不赞成
该特性已知有问题，并且计划进行修改。不要依赖该特性，使用它会造成警告，向后兼容也不会被考虑到。
```

```
Stability: 1 - Experimental
This feature was introduced recently, and may change
or be removed in future versions.  Please try it out and provide feedback.
If it addresses a use-case that is important to you, tell the node core team.
```

```
Stability: 2 - Unstable
The API is in the process of settling, but has not yet had
sufficient real-world testing to be considered stable. Backwards-compatibility
will be maintained if reasonable.
```

```
Stability: 3 - Stable
The API has proven satisfactory, but cleanup in the underlying
code may cause minor changes.  Backwards-compatibility is guaranteed.
```

```
Stability: 4 - API Frozen
This API has been tested extensively in production and is
unlikely to ever have to change.
```

```
Stability: 5 - Locked
Unless serious bugs are found, this code will not ever
change.  Please do not suggest changes in this area; they will be refused.
```

## JSON Output

    Stability: 1 - Experimental

Every HTML file in the markdown has a corresponding JSON file with the
same data.

This feature is new as of node v0.6.12.  It is experimental.
