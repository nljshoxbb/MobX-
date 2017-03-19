# MobX-
学习MobX和翻译文档

## 介绍
MoxB一直在尝试通过响应式编程让状态管理更简单和更易于扩展。MobX背后的原理非常简单：可以从应用程序状态中自动派生出任何内容，其中包括UI，数据序列化，服务器通信等。

React和MobX是一个强大的组合。React通过提供将其转换为可渲染组件树的机制来呈现应用程序状态。React提供了通过使用虚拟DOM机制,减少了昂贵的DOM操作，从而最佳地呈现出UI。MobX提供一个机制，即通过响应式虚拟关系状态映射，来最佳地同步应用程序状态与React组件，该关系仅在需要时更新，并且永远不会过时。

### 核心概念
MobX只有几个核心概念。以下代码片段可以使用[JSFiddle在线尝试](https://jsfiddle.net/mweststrate/wv3yopo0/)（或[不使用ES6和JSX](https://jsfiddle.net/rubyred/55oc981v/)）

## 可观察状态
MobX向现有的数据结构（如对象，数组和类实例）添加了可观察的功能。可以简单地通过使用`@observable`装饰器（ES.Next）注释类属性。
```

class Todo {
    id = Math.random();
    @observable title = "";
    @observable finished = false;
}

```

用`observable`就像将对象的属性转换为电子表格单元格。但是与电子表格不同，这些值不仅仅是原始值，还包括引用，对象和数组。您甚至可以[定义自己的](http://mobxjs.github.io/mobx/refguide/extending.html)可观察数据源

## 在ES5，ES6和ES.next环境中使用MobX
虽然`@`这个东西看起来陌生，这些是ES.next语法的装饰器。在MobX中，使用它们是完全可选的。有关如何使用的详细信息，请参阅[文档](http://mobxjs.github.io/mobx/best/decorators.html)。 MobX可以在任何ES5环境上运行，但是利用ES.next功能（如装饰器）是使用MobX时的最佳选择。

在ES5中，上面的代码段将如下所示：

```
function Todo() {
    this.id = Math.random()
    extendObservable(this, {
        title: "",
        finished: false
    })
}
```

## 计算出的值
使用MobX，您可以定义在修改相关数据时自动计算出派生的值。通过使用`@computed`装饰器或使用getter / setter函数时使用`（extend）Observable`.
```
class TodoList {
    @observable todos = [];
    @computed get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length;
    }
}
```

MobX将确保在添加todo或属性修改`finished `时，unfinishedTodoCount会自动更新。这种计算类似于电子表格程序中的公式，如MS 的Excel。只有当需要时会自动更新。

## 响应
响应有点类似于计算出的值。但响应不是产生一个新的值，而是做出相应的附加操作，例如打印到控制台，进行网络请求，增量更新React组件树以操作DOM等。

### React组件
如果你使用React，你可以把你的（无状态函数）组件转换为响应组件，只需在mobx-react包中的`observer`函数/装饰器添加到它们之前。
```
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import {observer} from "mobx-react";

@observer
class TodoListView extends Component {
    render() {
        return <div>
            <ul>
                {this.props.todoList.todos.map(todo =>
                    <TodoView todo={todo} key={todo.id} />
                )}
            </ul>
            Tasks left: {this.props.todoList.unfinishedTodoCount}
        </div>
    }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)

const store = new TodoList();
ReactDOM.render(<TodoListView todoList={store} />, document.getElementById('mount'));
```

`observer `将React（函数）组件转换为他们渲染数据派生所出来的。当使用MobX时，没有逻辑或纯渲染的组件。所有组件智能地渲染，但以固定的方式定义。MobX将确保组件总是在需要时重新渲染，并且会过滤没有操作的状态。因此，上面例子中的onClick处理程序将强制对TodoView进行渲染，并且如果未完成任务的数量已经改变，TodoListView也会重新渲染。但是，如果您删除`Tasks left`（或将其放入一个单独的组件），当勾选框时，TodoListView将不再重新渲染.你可以通过更改[JSFiddle](https://jsfiddle.net/mweststrate/wv3yopo0/)进行验证。

### 自定义响应
可以通过相应的情况，创建自定义响应函数，如`autorun`，`autorunAsync`或`when`。
例如，以下`autorun`将在`unfinishedTodoCount`更改时打印一条日志消息.
```
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})
```

## MobX会做什么反应?
为什么每次更改unfinishedTodoCount时都会打印一条新消息？答案是：MobX对在执行跟踪函数期间读取的任何现有具有observable属性做出响应。有关MobX如何确定哪些可观察对象需要做出响应的深入解释，请查看[understanding what MobX reacts to](https://github.com/mobxjs/mobx/blob/gh-pages/docs/best/react.md).

## actions
与许多通量框架不同，MobX不需要配置处理用户事件
* 这可以以类似于Flux的方式完成。
* 或者通过使用react全家桶来处理事件
* 或者通过简单地以最直接的方式处理事件，如上面的onClick处理程序所示

最后，一切都归结为：不知何故，状态应该更新。

更新状态后，`MobX`将以高效，无干扰的方式处理其余部分.如下所示，用几个简单的语句就足以自动更新用户界面。

不再需要调用类似dispatcher的操作。React组件只一个是状态机，最终状态将由MobX来管理。

```
store.todos.push(
    new Todo("Get Coffee"),
    new Todo("Write simpler code")
);
store.todos[0].finished = true;
```

尽管如此，MobX有一个可选的内置[actions](https://mobxjs.github.io/mobx/refguide/action.html)的概念。使用也有优势，比如它们将帮助您更好地构建代码，并很好的预测何时和何处修改状态。

## MobX：简单和可扩展
MobX虽然小，但可以用于状态管理。这使得MobX方法不仅简单，而且利于扩展。
