# MobX-
学习MobX3和翻译文档

# MobX概要

目前为止，这听起来有点奇怪，但是使用MboX来让你的应用改变为响应式，可以归结为三个步骤

## 1.定义状态，并使其可观察
将状态存储在任何你喜欢的数据结构中;对象，数组，类。循环数据结构，引用，这样都没有关系。只要确保想要观察的属性标记为`mobx`，那么你所标记的属性随时间的变化都会被记录下来。

```
import {observable} from 'mobx';

var appState = observable({
    timer: 0
});

```

## 2.创建一个响应状态变化的视图

在appState 中任何属性都可以被追踪观察; 你现在可以在appState中创建相关数据，并在数据更改时，引起视图的自动更新。 MobX将找到性能最优的方式来更新你的视图。 这个事实节省了大量的样板，提高你的效率。

一般来说，任何函数都可以成为观察其数据的反应视图，MobX可以应用于任何符合ES5的JavaScript环境。但是这里是一个以React组件形式的视图的（ES6）示例
```
import {observer} from 'mobx-react';

@observer
class TimerView extends React.Component {
    render() {
        return (<button onClick={this.onReset.bind(this)}>
                Seconds passed: {this.props.appState.timer}
            </button>);
    }

    onReset () {
        this.props.appState.resetTimer();
    }
};

React.render(<TimerView appState={appState} />, document.body);
```

## 3.修改状态

第三件事是修改状态。That is what your app is all about after all。 与许多其他框架不同，MobX不会决定如何做到这一点。 有最佳实践，但是需要记住的是：MobX帮助你以简单直接的方式做事情。
以下代码将每秒更改你的数据，并且UI将在需要时自动更新。 在更改状态的控制器函数或者应该更新的视图中，你可以不用定义显式关系。 因为使用observable装饰你的状态和视图足以让MobX检测所有关系。 下面是改变状态的两个例子：
```
appState.resetTimer = action(function reset() {
    appState.timer = 0;
});

setInterval(action(function tick() {
    appState.timer += 1;
}), 1000);

```
仅当在严格模式下使用MobX时才需要动作包装器（默认关闭）。 建议使用`action`，因为它将帮助您更好地结构化应用程序，并表达一个函数修改状态的意图。 它还自动应用事务以获得最佳性能

# 概念 & 原则

## 概念
MobX区分应用程序中的以下概念。你在之前的概要简述中看到了他们，但这里让我们更深入地了解他们

## 1.State
State是驱动应用程序的数据。通常有域特定的状态，例如一个待办事项列表，并有视图状态，如当前选择的元素。记住，状态就像保存着数据的电子表格

## 2.Derivations
可以从没有任何进一步交互的状态导出的任何事物都是属于派生，所以衍生以许多形式存在：
* 用户界面
* 派生数据，如剩余的todos数量。
* 后端集成，如发送更改到服务器

MobX  在派生的概念中与别的库有所不同:
* `Computed values`。这些是可以总是使用纯函数从当前可观察状态导出的值
* `Reactions`。`Reactions`是需要在状态改变时自动发生的副作用。这些是需要作为命令式和反应式编程之间的桥梁。或者使它更清楚，它们最终需要实现I / O。

刚开始使用MobX的人倾向于经常使用`reactions`。推荐是如果要根据当前状态创建一个值，请使用`computed`。

## 3. Actions
`Actions`是任何改变状态的代码段。用户事件，后端数据推送，预定事件等。操作类似于在电子表格单元格中输入新值的用户。
`Actions`可以在MobX中明确定义，以帮助你更清晰地构造代码。如果MobX在严格模式下使用，MobX将强制任何状态都不能被修改。

# 原则

MobX支持单向数据流，其中 `Actions` 更改状态，这反过来更新所有受影响的视图。


|  Action  |------->|  State  |------>|  Views  |


当状态改变时，所有派生出来的值都会自动更新。这样的结果可想而知，我们永远不可能观察到中间值的变化。
默认情况下，所有派生出来的值均同步更新。这意味着例如`Actions`可以在改变状态之后直接安全地检查计算值.
`Computed values`是惰性更新的。任何`computed value`将不会更新，直到需要副作用（I / O）。如果视图不再使用，它​​将被自动进行垃圾回收.
所有`Computed values`应为纯净的。他们不应该去改变`state`。

# Illustration
以下列表说明了上述概念和原则：
```
import {observable ,autorun} from 'mobx';

var todoStore = observable({
    //一些被观察的状态
    todos:[],

    //一个被派生出的值
    get completedCount (){
        return this.todos.filter(todo => todo.completed).length;
    }
})

//观察state的函数
autorun(function(){
   console.log("Completed %d of %d items",
        todoStore.completedCount,
        todoStore.todos.length
   );
})

/* 通过actions 修改state */
todoStore.todos[0] = {
    title: "Take a walk",
    completed: false
};
// -> 同步打印出 'Completed 0 of 1 items'

todoStore.todos[0].completed = true;
// -> 同步打印出 'Completed 1 of 1 items'


```
