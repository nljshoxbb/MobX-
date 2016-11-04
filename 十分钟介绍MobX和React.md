#十分钟介绍MobX 和 React(翻译，原文[Ten minute introduction to MobX and React](https://mobxjs.github.io/mobx/getting-started.html))
MobX是一个简单、可扩展和可测试的状态管理解决方案。这个十分钟的教程会教你所有重要的MobX的概念。MobX是一个独立的库，但是很多人把他和React结合使用，这个教程就是着重讲如何和React使用的。
##核心思想
状态对每一个应用来说都是核心的东西，如果没有很好的一个状态管理办法，将很容易产生不一致的状态，出现很多问题。比如与周围存在的局部变量不同步的状态等等。于是很多状态管理解决方案尝试去约束使用者，例如，使用不可变的状态。但是这样做产生了新问题，数据需要标准化，没有准确的参考，并且它无法使用原型大法。
MobX不会产生不一致的状态，这从根本上使得管理数据变得简单。实现这一点的方法很简单，确保可以从应用状态派生的所有内容都会自动派生。
从概念上讲，MobX会像电子表格一样处理您的应用程序。
![](/img/overview.png) 
* 1.首先，这里有一个应用的状态。对象、数组、元素，用这些图表示应用程序模型的引用关系。这些值就是程序的“数据单元”。
* 2.其次，他们具有派生功能。理论上说，任何值都可以从应用程序的状态中计算出来。这些派生或者是计算出来的值，他们可以来自简单的数字计算，也可以来自复杂的HTML可视化表示。如果把管理状态看作一张电子表格，那么这些状态就是电子表格中的图表和公式。
* 3.derivations和reactions非常相似。主要一个区别就是reactions的函数不会产生值。他们只是在自动执行某些程序。通常这是和I/O相关的。他们确保DOM的及时更新和正确自动的发起网络请求。
* 4.所有改变state的都在actions中发生。MobX会确保你通过actions来改变应用的状态，产生的所有改变由derivations和reactions来自动处理。同步没有干扰。

##一个简单的todo store ...
对理论有一定了解之后，可以写一些代码来加深印象。为了原创，我们从一个非常简单的ToDo存储开始。下面是一个非常直接的TodoStore，它维护着一个todos的集合。没有MobX参与
```
class TodoStore {
	todos = [];

	get completedTodosCount(){
		return this.todos.filter(
			todo=>todo.completed === true
			).length;
	}

	report(){
		if(this.todos.length === 0){
			return "<none>";
		}
		return 'Next todo:'+this.todos[0].task+'Progress:'+this.completedTodosCount/this.todos.length;
	}

	addTodo(task){
		this.todos.push({
				task:task,
				completed:false,
				assignee:null
			})
	}
}

const todoStore = new TodoStore();
```

我们只是创建了一个todoStore的实例和一个todos的集合。向todoStore中加入一些对象，确保在每一个改变之后调用todoStore.report方法，并且打印出来，出现明显变化。我们看到report一直只显示first task.这个例子看上去有点假，但是在下面中我们可以清楚看到MobX的依赖关系是动态跟踪的。
```
todoStore.addTodo("read MobX tutorial111");
console.log(todoStore.report());

todoStore.addTodo("try MobX");
console.log(todoStore.report());

todoStore.todos[0].completed = true;
console.log(todoStore.report());

todoStore.todos[1].task = "try MobX in own project";
console.log(todoStore.report());

todoStore.todos[0].task = "grok MobX tutorial";
console.log(todoStore.report());
```

结果
```
Next todo: "read MobX tutorial". Progress: 0/1
Next todo: "read MobX tutorial". Progress: 0/2
Next todo: "read MobX tutorial". Progress: 1/2
Next todo: "read MobX tutorial". Progress: 1/2
Next todo: "grok MobX tutorial". Progress: 1/2
```

##变成数据响应式

目前来看，这些代码没有什么特殊的地方。但如果我们不一定要显示的调用report这个方法也可以，我们是否可以在每次状态改变时都调用report?这样可以让我们避免在任何代码库中可能影响report的地方调用report而带来的错误。我们需要最新的数据将被打印出来，并且希望不被组织代码困扰。
幸好MobX可以帮我们解决这一个问题。自动执行的代码完全取决于状态。所以我们的report函数会自动更新，就像电子表格中的图表一样。为了实现这一点，TodoStore必须成为可被观察的，以便MobX进行跟踪正在改变的状态。同样的，completedTodosCount这个属性会从todo list中自动派生出来。通过使用@observable和@computed构造器，我们可以在一个对象中使用obsevable属性。

```
class ObservableTodoStore{
	@observable todos = [];
	@observable pendingRequests = 0;

	constructor(){
		mobx.autorun(()=>console.log(this.report));
	}

	@computed get completedTodosCount(){
		return this.todos.filter(
			todo=>todo.completed === true
			).length;
	}

	@computed get report(){
		if(this.todos.length === 0){
			return "<none>";
		}
		return 'Next todo:'+this.todos[0].task+'Progress:'+this.completedTodosCount/this.todos.length;
	}

	addTodo(task){
		this.todos.push({
				task:task,
				completed:false,
				assignee:null
			})
	}
}

const observableTodoStore = new ObservableTodoStore();
```
就是这样，我们用@observable标记出一些属性让MobX知道这些被标记的值会随时改变。计算属性可以用@computed来定义，用来识别导出的状态。
目前`pendingRequests`和`assignee`属性没有被使用，但会在之后的教程中使用。为了简洁，教程的所有代码均用es6书写，jsx和构造器。但是不用担心，所有MobX的构造器也可以在es5中使用。

在构造器中创造了一个打印`report`，并且被`autorun`包裹的小函数。Autorun 创建了一个响应，一旦被运行，在任何一个observable的数据改变都会自动重新运行这个函数。因为`report`使用的是observable的`todo`属性，他会在任何适当的时候检查。
```
observableTodoStore.addTodo("read MobX tutorial");
observableTodoStore.addTodo("try MobX");
observableTodoStore.todos[0].completed = true;
observableTodoStore.todos[1].task = "try MobX in own project";
observableTodoStore.todos[0].task = "grok MobX tutorial";
```

打印得到
```
Next todo: "read MobX tutorial". Progress: 0/1
Next todo: "read MobX tutorial". Progress: 0/2
Next todo: "read MobX tutorial". Progress: 1/2
Next todo: "grok MobX tutorial". Progress: 1/2
```
`report`并没有自动同步打印出中间的值。如果要仔细研究打印的东西，你可以看到第四行没有产生新的日志。因为`report`实际上没有例如重命名那样的改变，尽管返回的数据已经改变。从另一方面来看，改变第一个todo会更新report,从这时候起，name这个属性被report主动使用。这个例子充分说明，不但是todos的数组被观察并`autorun`,而且一些todo item中自有的属性也会被`autorun`。
<<<<<<< HEAD

##让react响应起来
好的，目前我们做了一个极其无聊的即时反映。现在开始做一个对用户操作进行反应的store。React Component 不会在store之外即时映射用户的操作。来自`mobx-react`的 `@observer`构造器解决React Componet在render方法中执行`autorun`,并自动异步保持状态。这与我们之前例子中做的`report`没什么不同。
下面清单定义了几个React component。在那里关于MobX的就是`@observer`构造器。这样能够确保每个组件在相关数据发生改变的时候可以各自更新呈现。你不需要再使用`setState`这个方法，也不必通过selectors或者高阶组件来配置应用中的props属性。基本上，所有组件变得很聪明。但是他们是声明的方式定义的。

```

@observer
class TodoList extends React.component {

	onNewTodo = ()=>{
		this.props.store.addTodo(prompt('Enter a new todo:','coffee piz'));	
	}

	render(){
	const store = this.props.store;
	return (
		<div>
			{store.report}
			<ul>
			{store.todos.map((todo,idx)=><TodoView todo={todo} key={idx}/>)}
			</ul>
			{store.pendingRequests > 0 ?
				<marquee>Loading</marquee>:
				null
			}
			<button onClick={this.onNewTodo}>New Todo</button>
			<small>(double-click a todo to edit)</small>
			<RenderCounter/>
		</div>
	)
	}
}

@observer
class TodoView extends React.component {
	onToggleCompleted = () = >{
		const todo = this.props.todo;
		todo.completed = !todo.completed;
	}

	onRename = ()=>{
		const todo = this.props.todo;
		todo.task = prompt('Task name',todo.task) || todo.task;
	}

	render(){
		const todo = this.props.todo;
		return(
			<li onDoubleClick={this.onRename}>
				<input 
					type="text"
					checked={todo.completed}
					onChange={this.onToggleCompleted}
					/>
				{todo.task}
				{todo.assignee
					?<small>{todo.assignee.name}</small>
					:null
				}
				<RenderCounter/>
			</li>
			)
	}
}

ReactDOM.render(
  <TodoList store={ observableTodoStore } />, 
  document.getElementById('reactjs-app')
);

const store = observableTodoStore;
store.todos[0].completed = !store.todos[0].completed;
store.todos[1].task = "Random todo " + Math.random();
store.todos.push({ task: "Find a fine cheese", completed: true });

```

MobX会自动派生和更新用户在应用中改变部分的状态。

##使用引用

目前为止，我们创建了可观察的对象（原型和对象），数组还有元素。你可能会有疑问，如何在MobX中控制引用呢？是否能把状态想象成一幅图？在上面的列表中，你可能会注意到在 todos有一个`assignee`属性。我们创建另一个store，并给assignee赋值

```
var peopleStore = mobx.observable([
    { name: "Michel" }, 
	{ name: "Me" } 
]);
observableTodoStore.todos[0].assignee = peopleStore[0];
observableTodoStore.todos[1].assignee = peopleStore[1];
peopleStore[0].name = "Michel Weststrate";
```

现在我们有两个独立的store.一个有people,一个有todos。为了从people store中分配一个`assignee`给一个people,我们只要分配一个引用就可以了。当这些放生改变的时候会通过`TodoView`自动被改变。通过使用MobX，不需要先规范好数据，然后也不用写确保组件及时更新的选择器。事实上，你把数据存在哪都没关系，只要对象都是可观察的，MobX会跟踪他们的状态变化。真正的js引用也会正常工作。如果他们在导出的时候相关，MobXh会自动跟踪他们。为了测试，你试着改变名字。

html的结构很简单
`<input onkeyup="peopleStore[1].name = event.target.value" />`


##异步操作

自从所有的东西都是从小程序状态中派生的，所以小程序并不担心状态在哪时候变化。这样使得创建异步操作变得很简单。只需按下面的按钮（多次）来模拟异步加载新的todo项目。
后面的代码真的很简单，我们从更新存储属性pendingRequests开始，让UI反映当前的加载状态。一旦加载完成，我们更新store的todos并再次减少pendingRequests计数器.只需将此代码段与之前的TodoList定义进行比较，即可了解如何使用pendingRequests属性。
```
observableTodoStore.pendingRequests++;
setTimeout(function() {
    observableTodoStore.addTodo('Random Todo ' + Math.random());
    observableTodoStore.pendingRequests--;
}, 2000);
```


##开发工具
mobx-react-devtools包提供了你在这个屏幕的右上角找到的devtools，可以在任何MobX + ReactJS应用程序中使用。单击第一个按钮将突出显示正在重新渲染的每个@observer组件。如果单击第二个按钮，然后单击预览中的一个组件，则会显示该组件的依赖关系树，以便您可以精确检查在任何给定时刻观察的数据。

##结论
就这样！没有样板。只是一些简单的声明性组件，形成我们的完整的UI。这些是完全从状态中派生，数据驱动性的。现在，您可以开始在自己的应用程序中使用mobx和mobx-react包。现在对目前为止讲到东西做简短摘要：
1.使用`@observable`装饰器或observable（对象或数组）函数使MobX跟踪对象。
2.`@computed`装饰器可以用于创建可以从状态自动计算出值的函数。
3.使用`autorun`来自动运行依赖于某个可观察状态的函数，这对记录，网络请求等很有用。
3.使用来自mobx-react包的@observer装饰器，使您的React组件真正响应。他们将自动和有效地更新，即使用于具有大量数据的大型复杂应用程序。

##MobX不是一个状态容器
人们经常使用MobX作为Redux的替代品。但请注意，MobX只是一个库来解决技术问题，而不是一个架构或甚至状态容器本身。在这个意义上，上面的例子是设计的，并且推荐使用适当的工程实践，如在方法中封装逻辑，在stroe或控制器中组织它们。
=======
>>>>>>> 2409a548f044258b9fe61211929a0c6f0609c571
