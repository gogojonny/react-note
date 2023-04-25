# useCallback

`useCallback`是一个React Hook.  让你在重绘时缓存函数定义

```react
const cachedFn = useCallback(fn, dependencies)
```

**参考**

* `useCallback(fn,dependencies)`

**使用**

* 跳过组件的重绘
* 从备忘录回调中更新状态
* 防止Effect过于频繁的执行
* 优化自定义hook

**疑难解答**

* 每次我的组件渲染时,`useCallback`返回一个不同的函数
* 我需要在每次循环创建的item中调用`useCallback`,但是这是不被允许的



## 参考

### `useCallback(fn, dependencies)`

在组件的顶部调用`useCallback`来在重绘期间缓存函数定义

```react
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```



**参数**

* `fn`: 你想缓存的函数.   这个函数可以接收任意多个参数并且可以返回任意类型的值.  React会在初次渲染期间将这个函数返回给你.  在下一次渲染时,  React会再次给你相同的函数如果依赖没有发生改变的情况下直到最后一次渲染.    否则,React会给你一个在渲染时你传递过来的函数,并且在这个函数可以被复用时将其储存.  React不会调用这个函数.  这个函数返回给你,  所以你可以决定何时在什么情况下调用它
* `dependencies`: 一个包含很多reactive value的参数列表, 这些value都是在`fn`中用得上的.   Reactive value包含props,  state 和所有直接在组件中声明的变量以及函数.  如果你的linter是被React配置的,  那么linter将会验证是否每一个reactive value都正确地使用. 依赖项列表必须具有恒定数量的项，并且像 [dep1， dep2， dep3] 一样内联编写。React将会比较每一个依赖值并用Object.is来判断是否相同

**返回值**

* 在初次渲染时,`useCallback`会返回你传入的`fn`
* 在接下来的渲染,  它要么返回一个已经存储到`fn`之中的函数从最后一次渲染中(如果依赖值没有发生改变)  要么返回一个在这次渲染时你传入的函数

**警告**

* `useCallback`是一个hook,  因此你只能在组件的顶部调用它或者在你的自定义hook中.  你不能在循环或者条件语句中调用它.  如果你需要这样做,那么将其冲相位一个组件并将state移入进去

* React不会抛弃缓存函数除非有一个具体的原因这么做

  例如,  在开发阶段,  React会在你编辑组件文件时抛弃缓存函数.  在开发阶段和产品成型时,  React会在你的组件挂载暂停时抛弃缓存函数.  在未来, React也许会添加更多特性来优化抛弃缓存函数这件事.  例如,  如果 React 将来添加对虚拟化列表的内置支持，那么抛弃那些滚出虚拟视口的缓存item将变得有意义.  如果你依赖`useCallback`作为性能优化将符合你的预期.  否则,  一个state变量或者ref也许更加合适.



## 使用

### 跳过重绘组件

当你做有关渲染的性能优化时， 你将有时需要缓存你向子组件传递的函数。让我们首先看看如何做到这一点吧，然后看看这在哪些情况下是有用的

为了缓存一个在组件之间的函数，将这个函数的定义用`useCallback`hook包裹起来

```react
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

你需要向其传入两个参数

1  一个你想在组件之间缓存的函数定义

2 一个依赖列表 包含在函数内部使用的所有组件内的值

在初次渲染时，React将会比较每一个依赖值，如果没有依赖值发生改变，那么`useCallback`就会返回和之前相同的函数。否则，它会返回在这次渲染时你传递给它的函数

**换句话说，`useCallback`缓存重新渲染之间的函数，直到其依赖值发生改变。**

让我们通过一个例子来看看这个是什么时候使用吧

假设你正在从`ProductPage`组件传递一个`handleSubmit`函数到`ShippingForm`组件.

```react
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

你会注意到当修改`theme`这个属性时,你的app会卡顿一会儿,  但是当你移除`ShippingForm /`从JSX中时在尝试修改theme,app就变快了.  所以尝试优化`shippingForm`组件是值得的

**默认情况下,当一个组件重绘时,React会递归地重绘其下所有的子元素**.   这也是为什么当`ProductPage`用不同的theme重绘时,`ShippingForm`组件也会重绘.  这对一些不需要太多计算去重绘的组件而言还比较好,  但是如果你发现重绘变慢. 你可以让`ShippingForm`跳过重绘过程 当它的props和上一次渲染时一致.  通过包裹memo

```react
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

通过这样的改变,**`ShippingForm`将跳过重绘 在props和上次渲染一致的情况下** 这时候缓存一个函数将变得十分重要 . 让我们假设你定义了一个没有用`useCallback`包裹的`handleSubmit`函数:

```react
function ProductPage({ productId, referrer, theme }) {
  // Every time the theme changes, this will be a different function...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* ... so ShippingForm's props will never be the same, and it will re-render every time */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

在JavaScript中,一个**`function () {}`**或者**`() => {}`**总是创造一个不同的函数.  这与{}对象字面量总是创建新的对象一样.  通常情况下, 这不会导致出现问题, 但是这意味着`ShippingForm`的props将永远都不一样,并且你的memo将不会工作.  这时候就需要`useCallback`来处理:

```react
function ProductPage({ productId, referrer, theme }) {
  // Tell React to cache your function between re-renders...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...so as long as these dependencies don't change...

  return (
    <div className={theme}>
      {/* ...ShippingForm will receive the same props and can skip re-rendering */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

通过用**`useCallback`**包裹起来函数,你就能确定在重绘之间两个函数相同. (直到依赖发生改变).  你不必将函数用**`useCallback`**包裹,  除非你这么做是为了一个具体的原因.  在这个例子中,原因就是你将组件用memo包裹来让他跳过重绘.  有更多的原因肯需要**`useCallback`**,  即需要进一步在页面中描绘。

---

**注意：**

**你应该只依赖`useCallback`作为性能优化**.   如果你的code没有它就无法工作,  寻找潜在的问题并解决它.  然后你可能在加上`useCallback`

---

**useCallback▁与▁useMemo▁有什么关系?**

你总是会看到`useCallback`在`useMemo`的旁边.  当你尝试优化一个组件时他们都是有用的. 他们都会缓存你传入的一些东西

```react
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // Calls your function and caches its result
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // Caches your function itself
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

他们的区别在于他们会缓存什么内容:

* `useMemo`缓存你调用的函数的result.   在这个案例中,它缓存了`computeRequirements(product)`的返回值,因此它不会发生改变,除非product发生改变.这个让你`requirements`对象不变来避免非必要的重绘.  在必要时,React将会调用你传入的函数在渲染期间来计算返回值
* `useCallback` 缓存函数自己.     不像`useMemo`,  他不会调用函数,相反它会缓存你提供的函数,以至于`handleSubmit`函数不会改变,除非`productId`和`referrer`发生了改变。这会让你传入的`handleSubmit`函数的组件不会做不必要的渲染。你的code不会执行直到用户submit了表单

如果你对`useMemo`很熟悉，你可能发现它有助于你思考`useCallback`

```react
// Simplified implementation (inside React)
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

useCallback实际上是对useMemo的封装

---



**你应该在任何地方都使用useCallback吗**

如果你的网站就像这个页面，并且大多数交互都是粗糙的，缓存就是不必要的。另一方面，如果你的app更像一个画画编辑器，并且大多数交互都是精细的，那么你也许发现缓存很有用。

用`useCallback`缓存函数只在下面几个情况下有用：

* 你将函数作为一个属性传入一个包裹了Memo的组件，你想要跳过多余的渲染。那么缓存将帮助你的组件不会做多余的渲染
* 你传入的函数之后会作为某些hook的依赖。例如，另一个包裹了`useCallback`的函数，或者这个函数要作为Effect的依赖

在其他情况下包裹`useCallback`不会有更多好处。同样的，包裹了`useCallback`也没有什么坏处。所以一些团队不会思考特例，而是尽可能的缓存。这样做会让代码变得可读性更差。同时，并不是所有的缓存都会生效：一个总会刷新的值足以破坏缓存。

注意到`useCallback`不会阻止创建函数。你也总是创建函数，但是React会忽略它并且给你一个缓存函数如果什么都没有发生变化的话。

在实践中，您可以通过遵循以下几个原则来创造大量不必要的缓存：

* 当一个组件视觉上包裹了另一个组件，且让它以一个JSX作为子元素。那么，如果包含者组件的state发生了改变，React就会知道子组件不需要重绘

* 多使用本地的props，尽量不要超出需求的使用状态提升。不要让state易发生改变，就像表单那样。

* 保持你的渲染逻辑纯粹。 如果重绘一个组件导致一个问题或者产生一些很明显的显示问题，这在组件中就是一个bug。修改这个bug而不是添加缓存
* 避免不必要的更新state的Effect。大多数显示问题都由于Effect导致的组件一遍又一遍的渲染的state更新链。
* 尝试从Effect中移除不必要的依赖。例如，将某些对象或函数移动到Effect内部或组件外部通常更简单，而不是缓存

如果某个具体的交互仍旧觉得迟缓，使用React开发者工具来看看那个组件最适合缓存，并且将缓存加入到需要的地方。上面的原则会是你的组件更容易debug并更容易理解，所以在任何情况下最好遵守他们。

但是如果如果组件加载足够快，那么就不需要添加缓存

请记住，您需要在生产模式下运行 React，禁用 React 开发人员工具，并使用与应用用户拥有的设备类似的设备，以便真实地了解实际减慢应用速度的因素。



### 从缓存回调中更新state

有时候，你需要从来自回调的上一次state中更新state

这个`handleAddToDo`函数将`todo`设置为依赖因为它会从todo上计算下一个todo

```react
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

通常你希望缓存一个依赖很少的函数。当你读取某个state只是为了计算下一个state时，你可以通过传入更新函数的方式来取消state作为依赖

```react
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ No need for the todos dependency
  // ...
```



### 防止Effect执行的太频繁

有时候你想在Effect内调用一个函数

```react

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...
```

这样做犯了一个错误，每一个reactive value必须被声明为一个Effect的依赖。然鹅，如果你将`createOptions作为依赖，那将会导致你的Effect不断的刷新

```react
 useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 Problem: This dependency changes on every render
  // ...
```

为了解决这个问题，你可以用`useCallback`包裹这个函数，然后再传入Effect中

```react
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Only changes when roomId changes

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ Only changes when createOptions changes
  // ...
```

这会确保Effect不会因为`createOption`函数因更换地址导致的改变从而触发更新。然鹅，最好移除掉函数依赖。将函数放入到Effect内部

```react
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ No need for useCallback or function dependencies!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only changes when roomId changes
  // ...
```

现在你的code就更加简单了并且不需要使用`useCallback`



### 优化自定义hook

如果你写了一个自定义hook并返回函数本身， 建议使用`useCallback`来包裹住返回值函数

```react
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

这确保了 Hook 的使用者可以在需要时优化他们自己的代码

## 疑难解答

### 每次我的组件渲染时，`useCallback`会返回一个不同的函数

确保你有一个依赖

如果你忘记加依赖，那么每一次渲染都会返回一个新的函数

```react
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Returns a new function every time: no dependency array
  // ...
```

下面是正确的做法

```react
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
```

如果这样仍不起效，那么就是有依赖在每次渲染时发生了变化，可以在控制台打印来消除bug

```react
  const handleSubmit = useCallback((orderDetails) => {
    // ..
  }, [productId, referrer]);

  console.log([productId, referrer]);
```

然后右键点击数组并将其存储到全局变量中。假设第一个存储到`temp1`且第二个存储到`temp2`，你可以使用浏览器打印台来比较两次是否一致

```react
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```



### 我需要为循环中的每个列表项调用 useCallback，但不允许这样做

假设`Chart`被包裹在`memo`中，你想让`Chart`当每次`ReportList`组件重绘时跳过重绘。然鹅，你不能在loop中调用`useCallback`

```react
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 You can't call useCallback in a loop like this:
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

你可以将`Chart`包裹在一个新的组件中

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
  // ✅ Call useCallback at the top level:
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```





















