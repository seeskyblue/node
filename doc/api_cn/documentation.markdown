# 关于本文

<!-- type=misc -->

本文的目的是从参考和概念的角度全面解释Node.js的应用程序接口（API），每个章节描述一个内置模块或高级概念。

属性类型、方法参数和事件处理的参数会被详细列在对应的标题下。

每个 `.html` 文件都有一个对应的内容相同的结构化的 `.json` 文件来表示。这个特性还是实验性质的，以便于集成开发工具（IDE）或其他工具希望能对本文档做操作。

每个 `.html` 文件和 `.json` 文件是基于对应的在node源代码中 `doc/api/` 文件夹中的 `.markdown` 文件生成的。使用 `tools/doc/generate.js` 程序进行生成。HTML模板文件是 `doc/template.html` 。

## 稳定度

<!--type=misc-->

贯穿全文，你会看到章节稳定度的标记。Node.js的API在某种程度上是变化中的，当他成长的时候，某些部分会相对其他部分更可靠。有些被证明过非常可靠，几乎不可能再被修改。也有些是新增的实验性的，或是已知有危险的并在重新设计的过程中。

稳定度描述如下：

```
稳定度：0 - 不赞成
该特性已知有问题，并且计划进行修改。不要依赖该特性，使用它会造成警告，向后兼容也不会被考虑到。
```

```
稳定度：1 - 实验性
该特性是最近引入的，可能在未来的版本中修改或删除。请使用之后提供反馈。
如果他正好适合一个对你来说重要的用户用例，请告诉node的核心团队。
```

```
稳定度：2 - 不稳定
该API正在稳定的进程中，但还未在实际环境中经过足够的测试验证为稳定。如果该特性合理会被进行向后兼容性维护。
```

```
稳定度：3 - 稳定
该API已被验证为符合要求，但代码中可能存在潜在的优化会造成细微调整。向后兼容性是可以保证的。
```

```
稳定度：4 - API冻结
该API在生产环境中被广泛测试过并看起来不再会改变。
```

```
稳定度：5 - 锁定
除非被发现严重的问题，否则不会修改代码。请勿建议在这些代码中进行修改，通常会被拒绝。
```

## JSON 输出

    稳定度: 1 - 实验性

每个通过markdown生成的HTML文件都对应一个具有相同数据的JSON文件。

该特性自node v0.6.12开始引入，目前还属于实验性。
