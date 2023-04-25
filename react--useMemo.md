# useMemo

`useMemo`是一个React Hook帮助你缓存重绘之间的渲染结果

**参考**

* `useMemo（calculateValue，dependencies）`

**使用**

* 跳过大量重计算
* 跳过重绘组件
* 缓存其他hook的依赖
* 缓存一个函数

**疑难解答**

* 每次重绘我的组件都会运行两次
* 我的`useMemo`原本返回一个对象，但是返回一个undefined
* 每次我的组件渲染时，在`useMemo`内的计算结果会再次执行
* 我需要为列表中每个item调用`useMemo`但是不被允许



## 参考

`useMemo（calculateValue，dependencies）`在组件顶部调用`useMemo`来缓存重绘时用的计算量

**参数**

* `calculateValue`这个函数会计算你想缓存的值，这个函数应该是纯函数且不接受参数，并且应该返回任意类型的值。react将在初次渲染时调用这个函数。并且在下一次渲染时，如果依赖值不发生改变，那么会返回相同的值。否则，它将调用这个函数，然后返回新的值
* `dependencies`一个依赖数组，每个值都用在`calculateValue`中

**返回值**

初次调用，`useMemo`返回调用`calculateValue`无参数的返回值，在下一次渲染时，它要么返回缓存值，要么调用函数返回新的值，并将这个值缓存

**警告**

* 所有的hook都不可以在循环或者条件语句中使用。如果你需要这样做，就把它包装成一个组件并移入state
* 严格模式会调用两次
* react不会抛弃缓存，除非有具体的原因。例如在开发阶段，当你编辑文件时，缓存会被丢弃。在开发和产品阶段，如果你的组件在初次挂载时有延迟，那么也会丢弃缓存。在未来react也许会增加更多特性来优化丢弃缓存这件事。



**Note：**
catch也叫memoization，这也是该hook叫做`useMemo`的原因



## 使用

### 跳过大量重计算

```react
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```



* 第一个参数是一个函数，且这个函数不需要参数，并且返回这个函数的返回值
* 一个依赖数组，当依赖发生改变时会再次执行第一个函数

`useMemo`会缓存计算结果并用于重绘，直到依赖值发生改变。

默认情况下，React将在每次重绘时会在执行组件体。例如`TodoList`更新state或者接收新的props，`filterTodos`函数将再次执行

```react
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

通常，这不会导致出现问题，因为大多数计算都很快。然鹅，如果你过滤或者转换一个大数组，你也许想跳过重执行过程，如果`todo`和`table`与上一次的渲染结果值一模一样。用`useMemo`包裹计算量就相当于更早地让你使用`visibleTodos`已经算好了



**Note**
你应该只依赖`useMemo`来做性能优化



### 跳过组建的重绘

在某些情况下，`useMemo`也能帮助子组件重绘时做性能优化。为了阐述这一点，假设`TodoList`组件传递`visibleValue`给子组件里的props。

```react
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

默认情况下，组件重绘时，react也会重绘其子组件.。如果想跳过子组件的重绘，可以使用`memo`包裹子组件

```react
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```



---



**深度探讨**

除了用`memo`包裹`List`，你也可以用`useMemo`来包裹`<List />`。

```react
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return (
    <div className={theme}>
      {children}
    </div>
  );
}
```

两者的行为是相同的，如果`visibleValue`不发生改变，那么`list`也不会重绘

一个JSX节点，例如`<List item={visibleTodos} />`是一个类似于`{ type: List, props: { items: visibleTodos } }`

的对象，创建对象很简单，但是React不知道是否对象的内容和上次相比是否发生了改变。这也是React会重绘的是组件的原因。

然鹅，React发现这次JSX与上一次渲染一致。他就不会尝试重绘组件，这是因为JSX是不可变的。JSX对象不可能随时间改变，所以react知道跳过重绘是安全的。但是，要使其起作用，节点实际上必须是同一个对象，而不仅仅是在代码中看起来相同。这就是 useMemo 在这个例子中所做的。

手动将 JSX 节点包装到 useMemo 中并不方便。例如，您不能有条件地执行此操作。这通常是为什么你会用备忘录包装组件而不是包装JSX节点。

---

### 缓存另一个hook的依赖

假设您有一个计算依赖于直接在组件主体中创建的对象

```react
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 Caution: Dependency on an object created in the component body
  // ...
```

依赖这样的对象会破坏缓存的意义。当组件重新呈现时，组件主体内的所有代码将再次运行。创建 `searchOptions` 对象的代码行也将在每次重新呈现时运行。由于 `searchOptions` 是 `useMemo` 调用的依赖项，并且每次都不同，因此 React 知道依赖项是不同的，并且每次都重新计算 `searchItems`。

要解决此问题，您可以在将 `searchOptions` 对象作为依赖项传递之前缓存它本身：

```react
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ Only changes when text changes

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ Only changes when allItems or searchOptions changes
  // ...
```

在上面的例子中，如果文本没有改变，`searchOptions` 对象也不会改变。但是，更好的解决方法是将 `searchOptions` 对象声明移到 `useMemo` 计算函数中：

```react
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ Only changes when allItems or text changes
  // ...
```

现在，您的计算直接依赖于文本（这是一个字符串，不会“意外”变得不同）。



### 缓存一个函数

假设表单组件包装在`memo`。你想把一个函数作为props传递给它

```react
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

正如 `{}` 创建不同的对象一样，函数声明（如` function() {}）`和表达式（如 `() => {}`）在每次重新渲染时都会产生不同的函数。就其本身而言，创建一个新函数不是问题。这不是可以避免的事情！然而，如果 Form 组件被缓存了，你可能想在没有 props 发生变化时跳过重新渲染它。总是不同的props会破坏缓存。

要使用useMemo记忆函数，您的计算函数必须返回另一个函数：

```react
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + product.id + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

这看起来很笨重！缓存函数很常见，以至于 React 有一个专门用于此的内置 Hook(也就是useCallback）。将函数包装到 useCallback 而不是 useMemo 中，以避免编写额外的嵌套函数：

```react
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + product.id + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

上面两个例子是完全等价的。 useCallback 的唯一好处是它可以让您避免在内部编写额外的嵌套函数。它什么都不做。阅读有关 useCallback 的更多信息。



## 疑难解答

### 我的计算在每次重新渲染时运行两次

在严格模式下，React 将调用你的一些函数两次而不是一次：

```react
function TodoList({ todos, tab }) {
  // This component function will run twice for every render.

  const visibleTodos = useMemo(() => {
    // This calculation will run twice if any of the dependencies change.
    return filterTodos(todos, tab);
  }, [todos, tab]);

  // ...
```

这是可预期的且不会破坏你的code

这种仅限开发的行为有助于您保持组件的纯洁性。React▁使用其中一个调用的结果,而忽略另一个调用的结果。只要您的组件和计算函数是纯净的,这不应该影响您的逻辑。但是,如果它们不小心不纯,这有助于您注意到并修复错误。

例如，这个不纯计算函数会改变您作为 prop 收到的数组：

```react
  const visibleTodos = useMemo(() => {
    // 🚩 Mistake: mutating a prop
    todos.push({ id: 'last', text: 'Go for a walk!' });
    const filtered = filterTodos(todos, tab);
    return filtered;
  }, [todos, tab]);
```

React 调用你的函数两次，所以你会注意到 TODO 被添加了两次。计算不应更改任何现有对象，但可以更改在计算过程中创建的任何新对象。例如，如果` filterTodos` 函数始终返回不同的数组，则可以改变该数组为：

```react
 const visibleTodos = useMemo(() => {
    const filtered = filterTodos(todos, tab);
    // ✅ Correct: mutating an object you created during the calculation
    filtered.push({ id: 'last', text: 'Go for a walk!' });
    return filtered;
  }, [todos, tab]);
```

### 我的 `useMemo` 调用应该返回一个对象，但返回未定义

这段code不会工作

```react
  // 🔴 You can't return an object from an arrow function with () => {
  const searchOptions = useMemo(() => {
    matchMode: 'whole-word',
    text: text
  }, [text]);
```

在 JavaScript 中，`（） => {` 启动箭头函数体，因此` { `大括号不是对象的一部分。这就是为什么它不返回对象并导致错误的原因。您可以通过添加括号`（{`和`}）`来修复它：

```react
  // This works, but is easy for someone to break again
  const searchOptions = useMemo(() => ({
    matchMode: 'whole-word',
    text: text
  }), [text]);
```

但是，这仍然令人困惑，并且太容易通过被人删除括号来打破

下面是正确做法

```react
  // ✅ This works and is explicit
  const searchOptions = useMemo(() => {
    return {
      matchMode: 'whole-word',
      text: text
    };
  }, [text]);
```

### 每次我的组件渲染时,useMemo▁中的计算都会重新运行

确保已将依赖项数组指定为第二个参数！

如果您忘记了依赖数组，useMemo 每次都会重新运行计算：

```react
function TodoList({ todos, tab }) {
  // 🔴 Recalculates every time: no dependency array
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  // ...
```

这是将依赖项数组作为第二个参数传递的更正版本:

```react
function TodoList({ todos, tab }) {
  // ✅ Does not recalculate unnecessarily
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
```

如果这没有帮助，那么问题是至少有一个依赖项与以前的渲染不同。您可以通过手动将依赖项记录到控制台来调试此问题：

```react
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  console.log([todos, tab]);
```

然后，您可以在控制台中右键单击来自不同重新渲染的数组，并为它们选择“存储为全局变量”。假设第一个被保存为 temp1，第二个被保存为 temp2，然后您可以使用浏览器控制台检查两个数组中的每个依赖项是否相同：

```react
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

当你发现某个依赖会破坏缓存时，要么也缓存它要么删除它

### 我需要为循环中的每个列表项调用 useMemo，但这是不允许的

假设`Chart`组件包装在`memo`中。你希望跳过每一个列表中的`Chart`当`ReportList`组件重绘时。然鹅，你不能在loop中调用`useMemo`。

```react
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 You can't call useMemo in a loop like this:
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```

正确的做法是将每个item用组件包裹起来

```react
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Call useMemo at the top level:
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

另外，你可以移除`useMemo`并且用`memo`来包裹`Report`。如果`item`的prop没有发生改变，`Report`将会跳过重绘，所以`Chart`也将会跳过重绘

```react
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  const data = calculateReport(item);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
});
```





























