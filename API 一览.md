# MobX Api参考

适用于MobX 3及更高版本。对于MobX 2，旧的文档仍然可以在github上

# 核心API

最重要的MobX api的。理解`observable`，`computed`，`reactions`和`actions`足以掌握MobX并在你的应用程序中使用它。

## observable

使用：
* `observable(value)`
* `@observable classProperty = value`

`observable`的值可以是原生js，引用，纯对象，类实例，数组和映射。 `observable（value）`是一个方便的重载，总是试图创建最佳匹配的`observable`类型。 你也可以直接创建所需的`observable`类型，请参见下文.

1.如果值是一个被[ES6 map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)包装的一个实例：将返回一个新的`Observable Map`。如果你不想只对特定条目的更改做出响应，而是对条目的添加或删除进行响应，那么可观察 map 将会非常有用。
2.如果值是一个数组，将返回一个新的Observable Array。
3.如果值是一个没有原型的对象，它的所有当前属性都将是可见的。请参阅可观察对象
4.如果值是具有原型，JavaScript原型或函数的对象，则将返回`Boxed Observable`。 MobX不会使用原型自动观察对象; 因为这是它的构造函数的责任。 在构造函数中使用`extendObservable`，或者在类定义中使用`@observable`。

这些规则看起来似乎很复杂，但你会注意到，在实践工作中，他们将会非常直观。一些注释：
* 要动态创建键值的对象，请使用`asMap`修饰符！因为只有对象上的最初存在的属性才会被观察，尽管新的属性可以使用`extendObservable`添加.
* 要使用`@observable`装饰器，请确保在转换器（babel或typescript）中启用装饰器。
* 默认情况下，使数据结构可观察是具有感染性的; 这意味着`observable`可以自动应用于数据结构包含的任何值，或者将来包含在数据结构中的值。 这种行为可以通过使用`modifiers`进行更改。

一些例子
```
const map = observable(asMap({ key: "value"}));
map.set("key", "new value");

const list = observable([1, 2, 4]);
list[2] = 3;

const person = observable({
    firstName: "Clive Staples",
    lastName: "Lewis"
});
person.firstName = "C.S.";

const temperature = observable(20);
temperature.set(25);
```

## @observable
可以在ES7或TypeScript类属性上使用的装饰器，以使它们可被观察。 `@observable`可以在实例字段和属性获取器中使用。这提供了可观察的细粒度控制，即对`object`的哪些部分进行观察。
```
import {observable} from "mobx";

class OrderLine {
    @observable price:number = 0;
    @observable amount:number = 1;

    constructor(price) {
        this.price = price;
    }

    @computed get total() {
        return this.price * this.amount;
    }
}
```

如果你的环境不支持装饰器或字段初始化器，`@observable key = value`;是`extendObservable(this, { key: value })`的语法糖

枚举：带有`@observable`的属性装饰器是可枚举的，但是在类原型而不是在类实例上定义.下面一些例子
```
const line = new OrderLine();
console.log("price" in line); // true
console.log(line.hasOwnProperty("price")); // false, the price _property_ is defined on the class, although the value will be stored per instance.
```

`@observable`装饰器可以与修饰符组合，如`asStructure`：
`@observable position = asStructure({ x: 0, y: 0})`

### 在你的编译器中启用装饰器
默认情况下，在使用TypeScript或Babel等待ES标准中的定义时，装饰器不受支持