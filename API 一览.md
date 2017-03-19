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

## (@)computed

`computed values`是可以从现有状态或其他计算值推导出的值。从概念上讲，它们与电子表格中的公式非常相似。你不能低估`computed values`，因为它们可以帮助你使实际可修改的状态尽可能小。此外，他们是高度优化，所以尽可能使用它们。
不要混淆`computed`与`autorun`。 它们都是响应变化时调用的表达式，但是如果你想在响应时产生一个可以被其他观察者使用的值，则使用`@computed`；而如果你不想产生一个新的值，并且想去实现一个效果，可以使用`autorun`,例如命令性副作用，如日志记录，网络请求等。

如果影响`computed values`任何值发生更改，则它将从你的状态中自动导出。 `computed values`在许多情况下可以被MobX优化，因为它们被假定为纯净的。 例如，如果前一个计算中使用的数据没有更改，计算属性将不会重新运行。 如果某个其他计算属性或响应中未使用计算属性，也不会重新运行。 在这种情况下，它将被暂停。

这种自动挂载非常方便。 如果`computed values`不再被观察，例如使用它的UI不再存在，MobX可以自动进行垃圾回收。 这不同于`autorun`的值，你必须自己处理它们。 它有时会让刚使用MobX人感到困惑，如果你创建一个`computed values`，但不在响应中的任何地方使用它，它也不会缓存其值和重新更多频率的计算。 但是在实际开发中，这是最好的默认值，如果你需要使用observe或keepAlive，你总是可以强制保持一个`computed values`。

请注意，`computed `属性不可枚举。它们也不能在继承链中被覆盖

### `@computed`
如果启用了装饰器，你可以对类属性的任何getter使用@computed装饰器来声明式创建`computed `属性。

```
import {observable, computed} from "mobx";

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

### `computed `修改
如果您的环境不支持装饰器，请结合使用computed（expression）修饰符和extendObservable / observable来引入新的`computed`属性。
`@computed get propertyName（）{}`相当于在构造函数调用`extendObservable（this，{propertyName：get func（）{}}）`。
```
import {extendObservable, computed} from "mobx";

class OrderLine {
    constructor(price) {
        extendObservable(this, {
            price: price,
            amount: 1,
            // valid:
            get total() {
                return this.price * this.amount
            },
            // also valid:
            total: computed(function() {
                return this.price * this.amount
            })
        })
    }
}


```
### `computed values`的设置
也可以为`computed values`定义设置器。注意，这些设置器不能直接用于改变`computed`属性的值，但它们可以用作“逆向”导出值。例如：

```
const box = observable({
    length: 2,
    get squared() {
        return this.length * this.length;
    },
    set squared(value) {
        this.length = Math.sqrt(value);
    }
});
```
使用装饰器
```
class Foo {
    @observable length: 2,
    @computed get squared() {
        return this.length * this.length;
    }
    set squared(value) { //this is automatically an action, no annotation necessary
        this.length = Math.sqrt(value);
    }
}

```
始终在getter之后定义setter，否则一些TypeScript版本已知声明两个具有相同名称的属性。

MobX 2.5.1或更高版本才提供设置器。

### `computed(expression)`作为函数

`computed`也可以直接作为函数调用。 就像`observable.box`（原始值）创建一个独立的`observable`。 对返回的对象使用`.get（）`以获取计算的当前值，或者使用`.observe（callback）`观察其更改。 这种形式的`computed`不是经常使用，但在某些情况下，你需要传递一个"boxed" computed value 它可能被证明是有用的

```
import {observable, computed} from "mobx";
var name = observable("John");

var upperCaseName = computed(() =>
    name.get().toUpperCase()
);

var disposer = upperCaseName.observe(change => console.log(change.newValue));

name.set("Dave");
// prints: 'DAVE'
```

### 配置`computed`
当使用`computed`作为修饰符或box时，它接受第二个选项参数和以下可选参数

* `name`:String，在spy和MobX devtools中使用的调试名称
* `context`:在表达式中使用
* `setter`:要使用setter函数。没有设置器，不可能为`computed value`分配新值。如果传递给`computed`的第二个参数是一个函数，则默认它是一个setter
* `compareStructural`:默认为false。 当为true时，表达式的输出在结果上与先前的值进行比较，然后通知任何观察者有关更改。 这确保了`computed`的观察者在没有发生结构改变的情况下不会返回一个新值。 这在使用点，矢量或颜色结构时非常有用。

### `@computed.struct`结构比较
`@computed`装饰器不接受参数。如果要创建执行结构比较的计算属性，请使用`@computed.struct`

### 关于错误处理的注意事项

如果`computed value `在其计算期间抛出异常，则此异常将捕获并在读取其值时重新抛出。 强烈建议始终抛出错误，以便保留原始堆栈跟踪。 例如：throw new Error（“Uhoh”）而不是throw“Uhoh”。 抛出异常不会中断跟踪，因此`computed values `可能从异常中恢复
```
const x = observable(3)
const y = observable(1)
const divided = computed(() => {
    if (y.get() === 0)
        throw new Error("Division by zero")
    return x.get() / y.get()
})

divided.get() // returns 3

y.set(0) // OK
divided.get() // Throws: Division by zero
divided.get() // Throws: Division by zero

y.set(2)
divided.get() // Recovered; Returns 1.5
```

## Autorun

`mobx.autorun`可以用在那些你想创建一个响应函数，本身永远不会有观察者。 这通常是当你需要从响应式转接到命令式代码时，例如对于日志记录，持久性或UI更新代码。 使用`autorun`时，所提供的函数将在其某个依赖关系发生更改时始终触发。 相比之下，`computed（function）`创建的函数只会重新评估它是否有观察者本身，否则它的值被认为是不相关的。 
一些建议：如果你有一个应该自动运行但不会产生新值的函数，请使用`autorun`,其他则使用`computed`。 `autorun`涉及引发效应，而不是产生新值。 如果字符串作为第一个参数传递给`autorun`，它将被用作调试名称。

传递给autorun的函数在调用时将接收一个参数，当前响应（自动运行）时，可用于在执行期间处理自动运行

就像`@observer`装饰器/函数一样，`autorun`只会观察在执行期间提供的函数所使用的数据
```
var numbers = observable([1,2,3]);
var sum = computed(() => numbers.reduce((a, b) => a + b, 0));

var disposer = autorun(() => console.log(sum.get()));
// prints '6'
numbers.push(4);
// prints '10'

disposer();
numbers.push(5);
// 不会打印任何东西, `sum'也不会被重新计算
```

### 异常处理
在`autorun`和所有其他类型反应中抛出的异常被捕获并记录到控制台，但不会传播回原始的导致代码。 这是为了确保一个异常中的反应不会阻止其他可能不相关的反应的预定执行。 这也允许反应从异常恢复; 抛出异常不会中断MobX进行的跟踪，因此，如果异常的原因被删除，随后的反应运行可能会再次正常完成。
可以通过调用反应的处理器上的`onError`处理程序来覆盖`Reactions`的默认日志行为。例:
```
const age = observable(10)
const dispose = autorun(() => {
    if (age.get() < 0)
        throw new Error("Age should not be negative")
    console.log("Age", age.get())
})

age.set(18)  // Logs: Age 18
age.set(-10) // Logs "Error in reaction .... Age should not be negative
age.set(5)   // Recovered, logs Age 5

dispose.onError(e => {
    window.alert("Please enter a valid age")
})

age.set(-5)  // Shows alert bolx
```

可以通过`extras.onReactionError（handler）`设置全局`onError`处理程序。这在测试或监控中很有用。


## @observer
`@observer`函数/装饰器可用于将ReactJS组件转换为响应的组件。 它将组件的render函数包装在`mobx.autorun`中，以确保在组件渲染期间使用的任何数据在发生更改时强制重新渲染。 它通过单独的`mobx-react`包提供

```
import {observer} from "mobx-react";

var timerData = observable({
    secondsPassed: 0
});

setInterval(() => {
    timerData.secondsPassed++;
}, 1000);

@observer class Timer extends React.Component {
    render() {
        return (<span>Seconds passed: { this.props.timerData.secondsPassed } </span> )
    }
});

React.render(<Timer timerData={timerData} />, document.body);
```

提示：当`observer`需要与其他装饰器或高阶组件组合时，请确保`observer`是最内层（首次应用的）装饰器;否则它可能什么都不做.

注意，使用`@observer`作为装饰器是可选的，`observer（class Timer ... {}）`实现完全相同.

### 在组件中取消引用值
MobX可以做很多，但它不能让原始值可观察（虽然它可以将它们包装在一个对象中，详情看` boxed observables`）。 但是对象的属性是可观察的。 这意味着`@observer`实际上反应了你取消引用一个值的事实。 所以在我们上面的例子中，如果Timer组件初始化如下，它不会反应
```
React.render(<Timer timerData={timerData.secondsPassed} />, document.body)
```
在这个片段中，只将当前值`secondsPassed`传递给`Timer`，它是不可变值0（所有primitives在JS中都是不可变的）。 这个数字在将来不会再改变，所以定时器永远不会更新。 它是将来会改变的属性`secondsPassed`，所以我们需要在组件中访问它。 或者换句话说：值需要通过引用传递，而不是通过值传递。

### ES5 support
在ES5环境中，观察器组件可以简单声明
```
observer(React.createClass({ .... 
```

### 无状态函数组件

上述定时器小部件还可以使用通过`observer`传递的无状态函数组件来编写
```
import {observer} from "mobx-react";

const Timer = observer(({ timerData }) =>
    <span>Seconds passed: { timerData.secondsPassed } </span>
);
```

### `observer`局部组件状态
就像普通类一样，你可以使用`@observable`装饰器在组件上引入`observable`属性。 这意味着你可以在组件中具有本地状态，而不需要使用React的冗长和强制的setState机制来管理它，但是它也同样拥有强大的功能。 响应状态将由render提取，但不会显式调用其他React生命周期方法，如`componentShouldUpdate`或`componentWillUpdate`。 如果你需要那些，只需使用正常的基于React状态的API。

```
import {observer} from "mobx-react"
import {observable} from "mobx"

@observer class Timer extends React.Component {
    @observable secondsPassed = 0

    componentWillMount() {
        setInterval(() => {
            this.secondsPassed++
        }, 1000)
    }

    render() {
        return (<span>Seconds passed: { this.secondsPassed } </span> )
    }
})

React.render(<Timer />, document.body)
```
有关使用observable本地组件状态的更多优点，请参见[为什么我停止使用setState的3个原因](https://medium.com/@mweststrate/3-reasons-why-i-stopped-using-react-setstate-ab73fc67a42e#.mn1e8jy0s)

### `observer`连接stores

`mobx-react`包还提供了`Provider`组件，可以用于使用React的上下文机制传递存储。 要连接到这些stores，请将stores名称以数组形式传递给`observer`，这将使stores提供类似props。 通过装饰器（`@observer（[“store”]）...`或函数`observer（[“store”]，React.createClass（{...``）`来使用
```
const colors = observable({
   foreground: '#000',
   background: '#fff'
});

const App = () =>
  <Provider colors={colors}>
     <app stuff... />
  </Provider>;

const Button = observer(["colors"], ({ colors, label, onClick }) =>
  <button style={{
      color: colors.foreground,
      backgroundColor: colors.background
    }}
    onClick={onClick}
  >{label}<button>
);

// later..
colors.foreground = 'blue';
// all buttons updated
```

有关更多信息，请参阅[mobx-react docs](https://github.com/mobxjs/mobx-react#provider-experimental)。

### 什么时候应用`observer`？
简单的经验法则是：渲染`observer`数据的所有组件。如果不想将组件标记为`observer`，例如为了减少通用组件包的依赖关系，请确保只传递纯数据。

使用`@observer`，不需要为了渲染的目的将“容器”组件与“展示”组件区分开。 它仍然是一个很好的分离的问题在哪里处理事件，请求等。所有组件在他们自己的依赖关系改变时才会负责更新。 它的开销是可忽略的，它确保每当你开始使用可观察的数据，组件将响应它。 有关更多详细信息，请参阅此线程