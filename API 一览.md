# MobX Api参考

适用于MobX 3及更高版本。对于MobX 2，旧的文档仍然可以在github上

# 核心API

最重要的MobX api的。理解`observable`，`computed`，`reactions`和`actions`足以掌握MobX并在你的应用程序中使用它。

## 创建observable
### `observable(value)`

使用：
* `observable(value)`
* `@observable classProperty = value`

`observable`的值可以是原生js，引用，纯对象，类实例，数组和映射。 `observable（value）`是一个方便的重载，总是试图创建最佳匹配的`observable`类型。 你也可以直接创建所需的`observable`类型，请参见下文.
1.如果值是一个包装是ES6map[]的一个实例：将返回一个新的Observable Map。如果您不想只对特定条目的更改做出反应，而是对条目的添加或删除，可观察映射非常有用。